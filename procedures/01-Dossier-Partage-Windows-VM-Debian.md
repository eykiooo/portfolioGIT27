---
title: Dossier partagé de Windows à une VM Debian
author: Samuel Mnassri
date: Janvier 2026
version: 1.0
---

# PROCÉDURE

## Configuration d'un Dossier Partagé entre Windows et une VM Debian

---

## 📋 TABLE DES MATIÈRES

1. Objectif
2. Prérequis
3. Étapes de configuration
4. Vérification et troubleshooting

---

## 1. OBJECTIF

Configurer un **dossier partagé** entre votre machine hôte **Windows** et une machine virtuelle **Debian** pour permettre l'échange de fichiers facilement.

**Avantage :** Travailler avec les fichiers directement depuis Windows et les utiliser immédiatement dans Debian sans transfert manuel.

---

## 2. PRÉREQUIS

- ✅ VirtualBox installé sur Windows
- ✅ Une machine virtuelle Debian fonctionnelle
- ✅ Guest Additions installés sur la VM Debian
- ✅ Accès administrateur sur Windows
- ✅ Accès root ou sudo sur Debian

---

## 3. ÉTAPES DE CONFIGURATION

### **ÉTAPE 1 : Préparer le dossier sur Windows**

1. Créez un dossier sur votre machine Windows (ex: `C:\SharedFolder`)
2. Clic droit → Propriétés → notez le chemin complet
3. Ce dossier sera partagé avec Debian

**💡 Conseil :** Placez-le dans vos Documents ou Bureau pour un accès facile

[Capture d'écran : Dossier créé sur Windows]

---

### **ÉTAPE 2 : Configurer le partage dans VirtualBox**

1. **Ouvrez VirtualBox** et sélectionnez votre VM Debian
2. Clic sur **Paramètres** (Settings)
3. Allez dans l'onglet **Dossiers partagés** (Shared Folders)

   [Capture d'écran : Menu Paramètres de VirtualBox]

4. Cliquez sur **Ajouter un dossier partagé** (icône +)

   [Capture d'écran : Fenêtre Dossiers partagés]

5. Remplissez les champs :
   - **Chemin du dossier** : `C:\SharedFolder` (votre chemin Windows)
   - **Nom du partage** : `shared` (ou le nom de votre choix)
   - ✅ Cochez **Montage automatique**
   - ✅ Cochez **Rendre permanent**

   [Capture d'écran : Formulaire d'ajout du dossier partagé]

6. Cliquez **OK**

---

### **ÉTAPE 3 : Installer Guest Additions (si pas déjà fait)**

1. Démarrez votre VM Debian
2. Dans le menu VirtualBox : **Périphériques** → **Insérer l'image CD Guest Additions**
3. Montez le CD :
   ```bash
   mkdir -p /media/cdrom
   mount /dev/cdrom /media/cdrom
   ```
4. Exécutez l'installateur :
   ```bash
   cd /media/cdrom
   sudo bash VBoxLinuxAdditions.run
   ```
5. Redémarrez la VM :
   ```bash
   sudo reboot
   ```

---

### **ÉTAPE 4 : Ajouter l'utilisateur au groupe vboxsf**

1. Connectez-vous sur Debian (ou ouvrez un terminal)
2. Exécutez :
   ```bash
   sudo usermod -aG vboxsf $(whoami)
   ```
3. **Déconnectez-vous puis reconnectez-vous** pour appliquer les changements (ou redémarrez)

---

### **ÉTAPE 5 : Vérifier et accéder au dossier partagé**

1. Ouvrez un terminal Debian
2. Listez les dossiers montés :
   ```bash
   ls -la /media/
   ```
3. Vous devriez voir votre dossier partagé (ex: `/media/sf_shared`)

4. Accédez-y :
   ```bash
   cd /media/sf_shared
   ls -la
   ```

   [Capture d'écran : Terminal avec dossier monté]

---

## 4. VÉRIFICATION ET TROUBLESHOOTING

### ✅ Dossier monté avec succès ?

```bash
mount | grep vboxsf
```

Vous devriez voir une ligne du type :
```
shared on /media/sf_shared type vboxsf (...)
```

---

### ⚠️ **Problème 1 : Dossier non accessible**

**Symptôme :** `/media/` ne contient pas le dossier partagé

**Solution :**
1. Vérifiez que Guest Additions est bien installé
2. Montez manuellement :
   ```bash
   sudo mount -t vboxsf shared /mnt/shared
   ```
3. Vérifiez les permissions :
   ```bash
   ls -la /mnt/shared
   ```

---

### ⚠️ **Problème 2 : Permission refusée (Permission denied)**

**Symptôme :** Impossible de créer/modifier des fichiers

**Solution :**
```bash
# Vérifiez que vous êtes dans le groupe vboxsf
id

# Si vboxsf n'apparaît pas, réexécutez :
sudo usermod -aG vboxsf $(whoami)
sudo reboot
```

---

### ⚠️ **Problème 3 : Dossier ne se monte pas automatiquement**

**Solution :**
1. Ajoutez une entrée dans `/etc/fstab` :
   ```bash
   sudo nano /etc/fstab
   ```
2. Ajoutez à la fin :
   ```
   shared  /media/sf_shared  vboxsf  defaults,uid=1000,gid=1000  0  0
   ```
3. Sauvegardez (Ctrl+O, Entrée, Ctrl+X)
4. Testez :
   ```bash
   sudo mount -a
   ```

---

## 📝 RÉSUMÉ

Vous avez configuré un **dossier partagé bidirectionnel** entre Windows et Debian. Vous pouvez maintenant :

- 📂 Créer/modifier des fichiers sous Windows et les utiliser immédiatement dans Debian
- 📂 Créer/modifier des fichiers dans Debian et les voir apparaître dans Windows
- 🔄 Synchroniser vos données sans transfert manuel

---

**Version :** 1.0  
**Date :** Janvier 2026  
**Auteur :** Samuel Mnassri

---
