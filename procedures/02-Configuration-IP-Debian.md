---
title: Configuration IP Debian
author: Samuel Mnassri
date: Janvier 2026
version: 1.0
---

# PROCÉDURE

## Configuration d'une Adresse IP sur Debian

---

## 📋 TABLE DES MATIÈRES

1. Objectif
2. Prérequis
3. Étapes de configuration (Netplan)
4. Étapes de configuration (ifupdown)
5. Vérification et troubleshooting

---

## 1. OBJECTIF

Configurer une **adresse IP statique ou dynamique** sur une machine Debian pour assurer une **connectivité réseau stable**.

**Cas d'usage :**
- IP statique : serveurs, services réseau
- IP dynamique (DHCP) : machines clients flexibles

---

## 2. PRÉREQUIS

- ✅ Debian installé et fonctionnel
- ✅ Accès root ou sudo
- ✅ Interface réseau activée
- ✅ Accès à un terminal
- ✅ Connexion à un réseau (Ethernet ou virtuel)

---

## 3. ÉTAPES DE CONFIGURATION (NETPLAN - Debian 10+)

### **ÉTAPE 1 : Identifier votre interface réseau**

1. Ouvrez un terminal
2. Listez les interfaces disponibles :
   ```bash
   ip link show
   ```
   
   Vous verrez quelque chose comme :
   ```
   1: lo: <LOOPBACK,UP,LOWER_UP>
   2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
   3: enp0s3: <BROADCAST,MULTICAST>
   ```

   [Capture d'écran : Résultat de la commande ip link show]

3. Notez le nom de votre interface (ex: `eth0`, `enp0s3`, `ens33`)

---

### **ÉTAPE 2 : Vérifier la configuration actuelle**

```bash
ip addr show
```

Vous verrez les adresses IP actuelles de chaque interface.

[Capture d'écran : Résultat de ip addr show]

---

### **ÉTAPE 3 : Configurer l'IP statique avec Netplan**

1. Ouvrez le fichier de configuration Netplan :
   ```bash
   sudo nano /etc/netplan/01-netcfg.yaml
   ```
   
   *(Si le fichier n'existe pas, créez-le)*

2. **Pour une IP STATIQUE**, entrez :
   ```yaml
   network:
     version: 2
     renderer: networkd
     ethernets:
       eth0:
         dhcp4: no
         addresses:
           - 192.168.1.100/24
         gateway4: 192.168.1.1
         nameservers:
           addresses: [8.8.8.8, 8.8.4.4]
   ```

   [Capture d'écran : Fichier netplan avec IP statique]

   **À adapter :**
   - `eth0` : remplacez par votre interface
   - `192.168.1.100` : votre IP souhaitée
   - `192.168.1.1` : adresse de la passerelle (routeur)
   - `8.8.8.8` : serveurs DNS (ou vos DNS préférés)

3. **Pour une IP DYNAMIQUE (DHCP)**, entrez :
   ```yaml
   network:
     version: 2
     renderer: networkd
     ethernets:
       eth0:
         dhcp4: true
   ```

4. Sauvegardez :
   - Appuyez sur **Ctrl+O** → **Entrée**
   - Appuyez sur **Ctrl+X**

---

### **ÉTAPE 4 : Appliquer la configuration**

```bash
sudo netplan apply
```

ou

```bash
sudo netplan generate
sudo systemctl restart networking
```

[Capture d'écran : Terminal après netplan apply]

---

### **ÉTAPE 5 : Vérifier la configuration**

```bash
ip addr show eth0
```

Vous devriez voir votre nouvelle adresse IP :
```
inet 192.168.1.100/24 brd 192.168.1.255 scope global eth0
```

[Capture d'écran : Nouvelle configuration IP]

---

## 4. ÉTAPES DE CONFIGURATION (IFUPDOWN - Debian 9 et antérieur)

### **Alternative : Configuration avec `/etc/network/interfaces`**

1. Ouvrez le fichier de configuration :
   ```bash
   sudo nano /etc/network/interfaces
   ```

2. **Pour une IP STATIQUE** :
   ```bash
   auto eth0
   iface eth0 inet static
       address 192.168.1.100
       netmask 255.255.255.0
       gateway 192.168.1.1
       dns-nameservers 8.8.8.8 8.8.4.4
   ```

3. **Pour DHCP** :
   ```bash
   auto eth0
   iface eth0 inet dhcp
   ```

4. Sauvegardez et redémarrez le service réseau :
   ```bash
   sudo systemctl restart networking
   ```

---

## 5. VÉRIFICATION ET TROUBLESHOOTING

### ✅ **Configuration réussie ?**

Exécutez :
```bash
ping 8.8.8.8
```

Si vous recevez des réponses, la connexion est établie ✅

[Capture d'écran : Ping réussi]

---

### ⚠️ **Problème 1 : Pas de connectivité**

**Symptôme :** Impossible de pinguer une adresse

**Solution :**
1. Vérifiez que l'interface est active :
   ```bash
   ip link show eth0
   ```
   Doit afficher `UP`

2. Si c'est `DOWN`, activez-la :
   ```bash
   sudo ip link set eth0 up
   ```

3. Redémarrez le service réseau :
   ```bash
   sudo systemctl restart networking
   ```

---

### ⚠️ **Problème 2 : Erreur "Permission denied" lors de la modification**

**Solution :**
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Utilisez `sudo` pour éditer le fichier.

---

### ⚠️ **Problème 3 : Changement IP ne s'applique pas**

**Solution :**
1. Vérifiez la syntaxe du fichier YAML :
   ```bash
   sudo netplan validate
   ```

2. Appliquez avec force :
   ```bash
   sudo netplan apply --debug
   ```

3. Si cela ne fonctionne pas, redémarrez le système :
   ```bash
   sudo reboot
   ```

---

### ⚠️ **Problème 4 : DNS ne fonctionne pas**

**Symptôme :** Impossible de résoudre les noms de domaine

**Solution :**
1. Vérifiez le fichier `/etc/resolv.conf` :
   ```bash
   cat /etc/resolv.conf
   ```

2. Modifier les DNS dans Netplan :
   ```yaml
   nameservers:
     addresses: [1.1.1.1, 1.0.0.1]
   ```

3. Appliquez :
   ```bash
   sudo netplan apply
   ```

---

## 📝 RÉSUMÉ

Vous avez configuré une **adresse IP** sur votre machine Debian (statique ou dynamique). Votre système est maintenant **connecté au réseau**.

**Commandes utiles à mémoriser :**
```bash
ip addr show                          # Voir toutes les IPs
ip link show                          # Voir les interfaces
ping 8.8.8.8                          # Tester la connectivité
sudo netplan apply                    # Appliquer les changements
```

---

**Version :** 1.0  
**Date :** Janvier 2026  
**Auteur :** Samuel Mnassri

---
