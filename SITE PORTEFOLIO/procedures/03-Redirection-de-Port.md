---
title: Redirection de Port
author: Samuel Mnassri
date: Janvier 2026
version: 1.0
---

# PROCÉDURE

## Configuration de la Redirection de Port sur Debian

---

## 📋 TABLE DES MATIÈRES

1. Objectif
2. Prérequis
3. Concepts fondamentaux
4. Étapes de configuration avec iptables
5. Étapes de configuration avec UFW
6. Vérification et troubleshooting

---

## 1. OBJECTIF

Configurer une **redirection de port** sur une machine Debian pour :
- Rediriger le trafic d'un port vers un autre
- Exposer un service interne sur un port externe
- Mettre en place une **translation d'adresses (NAT)**

**Exemple :** Rediriger le port 8080 vers le port 80 pour accéder à un service web.

---

## 2. PRÉREQUIS

- ✅ Debian installé et fonctionnel
- ✅ Accès root ou sudo
- ✅ Connaissance des ports TCP/UDP
- ✅ Firewall (iptables ou UFW)
- ✅ Un service actif sur le port cible
- ✅ Terminal ouvert

---

## 3. CONCEPTS FONDAMENTAUX

### **Qu'est-ce qu'une redirection de port ?**

La redirection de port permet de **mapper un port à un autre** sur la même machine ou une autre.

**Exemple visuel :**

```
Connexion externe : 192.168.1.100:8080
           ↓
    [Redirection]
           ↓
Service interne : localhost:80
```

### **Types de redirection :**

| Type | Description | Exemple |
|------|-------------|---------|
| **Port vers Port** | Même machine, ports différents | 8080 → 80 |
| **IP vers Port** | Vers une autre machine du réseau | 192.168.1.50:443 → localhost:8443 |
| **Port externe vers IP interne** | Permet d'accéder à un service sur le réseau | 0.0.0.0:8080 → 192.168.1.100:80 |

---

## 4. ÉTAPES DE CONFIGURATION AVEC IPTABLES

### **ÉTAPE 1 : Vérifier les règles iptables actuelles**

```bash
sudo iptables -t nat -L -n
```

[Capture d'écran : Résultat de iptables -L]

Cela affiche toutes les règles NAT actuelles.

---

### **ÉTAPE 2 : Ajouter une règle de redirection**

**Pour rediriger le port 8080 vers le port 80 :**

```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
```

**Explication :**
- `-t nat` : table NAT
- `-A PREROUTING` : ajouter une règle au chaîn PREROUTING
- `-p tcp` : protocole TCP
- `--dport 8080` : port destination (celui qu'on reçoit)
- `--to-port 80` : rediriger vers ce port

[Capture d'écran : Commande exécutée]

---

### **ÉTAPE 3 : Vérifier la règle ajoutée**

```bash
sudo iptables -t nat -L PREROUTING -n
```

Vous devriez voir :
```
REDIRECT   tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 redir ports 80
```

---

### **ÉTAPE 4 : Rendre la règle persistante**

⚠️ **Important :** Les règles iptables disparaissent après un redémarrage !

**Option A : Installer iptables-persistent**

```bash
sudo apt update
sudo apt install iptables-persistent
```

Lors de l'installation, choisissez **Yes** pour sauvegarder les règles actuelles.

[Capture d'écran : Installation iptables-persistent]

---

**Option B : Créer un script de démarrage**

1. Créez un fichier script :
   ```bash
   sudo nano /etc/init.d/iptables-rules.sh
   ```

2. Ajoutez :
   ```bash
   #!/bin/bash
   iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
   ```

3. Rendez-le exécutable :
   ```bash
   sudo chmod +x /etc/init.d/iptables-rules.sh
   ```

4. Enregistrez-le comme service :
   ```bash
   sudo update-rc.d iptables-rules.sh defaults
   ```

---

### **ÉTAPE 5 : Tester la redirection**

```bash
# Sur la machine Debian
curl http://localhost:8080

# Depuis une autre machine (remplacez par votre IP)
curl http://192.168.1.100:8080
```

Si le service sur le port 80 répond, la redirection fonctionne ✅

[Capture d'écran : Résultat du curl]

---

## 5. ÉTAPES DE CONFIGURATION AVEC UFW

**UFW (Uncomplicated Firewall)** est plus simple qu'iptables.

### **ÉTAPE 1 : Installer et activer UFW**

```bash
sudo apt install ufw
sudo ufw enable
```

[Capture d'écran : UFW activé]

---

### **ÉTAPE 2 : Configurer la redirection dans UFW**

1. Ouvrez le fichier de configuration UFW :
   ```bash
   sudo nano /etc/ufw/before.rules
   ```

2. **Avant la ligne `*filter`**, ajoutez :
   ```
   *nat
   :PREROUTING ACCEPT [0:0]
   -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
   COMMIT
   ```

   [Capture d'écran : Fichier before.rules modifié]

3. Sauvegardez (Ctrl+O → Entrée → Ctrl+X)

---

### **ÉTAPE 3 : Autoriser le trafic entrant**

```bash
sudo ufw allow 8080/tcp
sudo ufw allow 80/tcp
```

[Capture d'écran : Règles UFW ajoutées]

---

### **ÉTAPE 4 : Appliquer les changements**

```bash
sudo ufw reload
```

---

### **ÉTAPE 5 : Vérifier les règles**

```bash
sudo ufw status verbose
```

---

## 6. VÉRIFICATION ET TROUBLESHOOTING

### ✅ **La redirection fonctionne ?**

```bash
# Test local
curl http://localhost:8080

# Test distant (remplacez l'IP par la vôtre)
curl http://192.168.1.100:8080

# Vérifier les connexions actives
sudo netstat -tlnp | grep 8080
```

[Capture d'écran : Test réussi]

---

### ⚠️ **Problème 1 : "Connexion refusée"**

**Symptôme :** Impossible de se connecter au port 8080

**Solutions :**
1. Vérifiez que le service écoute sur le port 80 :
   ```bash
   sudo netstat -tlnp | grep 80
   ```

2. Vérifiez la règle iptables :
   ```bash
   sudo iptables -t nat -L PREROUTING -n
   ```

3. Si manquante, réappliquez-la :
   ```bash
   sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
   ```

---

### ⚠️ **Problème 2 : Règles perdues après redémarrage**

**Solution :**
```bash
# Vérifiez que iptables-persistent est installé
dpkg -l | grep iptables-persistent

# Si absent, installez-le
sudo apt install iptables-persistent

# Sauvegardez les règles actuelles
sudo iptables-save | sudo tee /etc/iptables/rules.v4
sudo ip6tables-save | sudo tee /etc/iptables/rules.v6
```

---

### ⚠️ **Problème 3 : Le firewall bloque le trafic**

**Symptôme :** Redirection configurée mais inaccessible

**Solution :**
```bash
# Avec iptables
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
sudo iptables -A FORWARD -p tcp --dport 80 -j ACCEPT

# Avec UFW
sudo ufw allow 8080/tcp
sudo ufw allow 80/tcp
```

---

### ⚠️ **Problème 4 : Impossible d'utiliser les ports < 1024**

⚠️ **Les ports < 1024 nécessitent les droits root/sudo**

**Solution :**
```bash
# Exécutez en tant que sudo
sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8443
```

---

## 📝 RÉSUMÉ

Vous avez configuré une **redirection de port** sur Debian. Le trafic reçu sur un port est maintenant **redirigé vers un autre port**.

**Commandes essentielles :**
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
sudo iptables -t nat -L PREROUTING -n
sudo netstat -tlnp | grep 8080
curl http://localhost:8080
```

**Pour rendre persistant :**
```bash
sudo apt install iptables-persistent
```

---

**Version :** 1.0  
**Date :** Janvier 2026  
**Auteur :** Samuel Mnassri

---
