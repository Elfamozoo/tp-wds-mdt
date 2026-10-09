# Déploiement automatique de Windows avec MDT et WDS

## 🧠 Objectif

Déployer un système d’exploitation personnalisé (Windows 10, 11, Server…) via MDT (Microsoft Deployment Toolkit) intégré au serveur WDS, avec :

- Applications préinstallées (ex : VLC)
- Paramètres automatisés
- Intégration automatique au domaine
- Installation silencieuse (sans interaction)
- Standardisation sur tout le parc informatique
## Sommaire

- [🧠 Objectif](#-objectif)
- [🧱 Composants nécessaires](#-composants-nécessaires)
- [📚 Définitions des termes clés](#-définitions-des-termes-clés)
- [1️ Installer les prérequis](#1️-installer-les-prérequis)
  - [a. Outils à installer sur le serveur](#a-outils-à-installer-sur-le-serveur)
- [2️ Créer l’environnement MDT](#2️-créer-lenvironnement-mdt)
  - [a. Créer le partage de déploiement](#a-créer-le-partage-de-déploiement)
- [3️ Importer l’image .WIM de Windows](#3️-importer-limage-wim-de-windows)
  - [a. Ajouter un système d’exploitation](#a-ajouter-un-système-dexploitation)
- [4️ Ajouter une application (ex : VLC)](#4️-ajouter-une-application-ex--vlc)
  - [a. Clic droit sur **Applications** → **New Application**](#a-clic-droit-sur-applications--new-application)
- [5️ Créer une Task Sequence](#5️-créer-une-task-sequence)
  - [a. Nouvelle séquence de tâches](#a-nouvelle-séquence-de-tâches)
- [6️ Lier l’application à la Task Sequence](#6️-lier-lapplication-à-la-task-sequence)
- [7️ Personnaliser les fichiers de réponse](#7️-personnaliser-les-fichiers-de-réponse)
  - [a. CustomSettings.ini](#a-customsettingsini)
  - [b. Bootstrap.ini](#b-bootstrapini)
- [8️ Générer les images personnalisées](#8️-générer-les-images-personnalisées)
  - [a. Mise à jour du partage MDT](#a-mise-à-jour-du-partage-mdt)
  - [b. Récupérer les images](#b-récupérer-les-images)
- [9️ Ajouter les images à WDS](#9️-ajouter-les-images-à-wds)
  - [a. Dans la console **WDS**](#a-dans-la-console-wds)
- [10 Déployer sur une machine cliente](#10-déployer-sur-une-machine-cliente)
- [✅ Résultat final](#-résultat-final)

## 🧱 Composants nécessaires

| Élément | Rôle |
|---|---|
| WDS | Permet aux clients de démarrer via le réseau (PXE) |
| MDT | Gère les images Windows, automatise l’installation |
| AD DS (Active Directory) | Gère les comptes utilisateurs et les ordinateurs (facultatif) |
| DHCP | Fournit automatiquement des adresses IP |
| Windows ADK | Fournit les outils de déploiement (Windows PE, etc.) |
| Images .WIM | Contiennent le système à déployer (install.wim) et le démarrage (boot.wim) |

## 📚 Définitions des termes clés

| Terme | Définition simple |
|---|---|
| PXE | Démarrage réseau sans clé USB (charge une image depuis le serveur) |
| WIM | Format d’image Windows (.wim = Windows Imaging Format) |
| Sysprep | Prépare un Windows pour être copié sur d’autres machines |
| Task Sequence | Suite d’étapes automatisées pendant l’installation |
| LiteTouchPE_x64.wim | Image de démarrage créée par MDT pour lancer l’installation |
| msiexec /quiet | Installation silencieuse d’un logiciel sans intervention |
| Bootstrap.ini / CustomSettings.ini | Fichiers de configuration qui contrôlent les options du déploiement |
| USMT | Outil qui permet de transférer les profils utilisateurs lors d’une migration (optionnel) |

## 1️ Installer les prérequis

### a. Outils à installer sur le serveur

- .NET Framework (La version diffère selon le serveur utilisé)
- Windows ADK :

![image-01.png](assets/image-01.png)

![image-02.png](assets/image-02.png)

- Windows PE Add-on for ADK

![image-03.png](assets/image-03.png)

![image-04.png](assets/image-04.png)

- Microsoft Deployment Toolkit (MDT)

![image-05.png](assets/image-05.png)

![image-06.png](assets/image-06.png)

## 2️ Créer l’environnement MDT

### a. Créer le partage de déploiement

- Ouvrir Deployment Workbench

![image-07.png](assets/image-07.png)

- Clic droit sur Deployment Shares → New Deployment Share

![image-08.png](assets/image-08.png)

- Emplacement : C:\DeploymentShare

![image-09.png](assets/image-09.png)

- Share name : DeploymentShare$

![image-10.png](assets/image-10.png)

- Nom affiché : MDT Deployment Share

![image-11.png](assets/image-11.png)

- Suivre l’assistant, décocher les options inutiles

![image-12.png](assets/image-12.png)

![image-13.png](assets/image-13.png)

## 3️ Importer l’image .WIM de Windows

### a. Ajouter un système d’exploitation

- Clic droit sur Operating Systems → Import Operating System

![image-14.png](assets/image-14.png)

- Choisir : Custom image files

![image-15.png](assets/image-15.png)

- Dossier contenant le fichier install.wim

![image-16.png](assets/image-16.png)

- Dossier de destination : installperso

![image-17.png](assets/image-17.png)

- Valider

![image-18.png](assets/image-18.png)

![image-19.png](assets/image-19.png)

## 4️ Ajouter une application (ex : VLC)

### a. Clic droit sur **Applications** → **New Application**

- Choisir : Application with source files

![image-20.png](assets/image-20.png)

- Nom : VLC

![image-21.png](assets/image-21.png)

- Source : dossier contenant vlc.msi

![image-22.png](assets/image-22.png)

- Commande de lancement :

```
msiexec.exe /i vlc.msi /quiet /norestart /qn
```

![image-23.png](assets/image-23.png)

## 5️ Créer une Task Sequence

### a. Nouvelle séquence de tâches

- Clic droit sur Task Sequences → New Task Sequence

![image-24.png](assets/image-24.png)

- ID : WIN10-CUSTOM
- Nom : Installation Windows 10 avec VLC

![image-25.png](assets/image-25.png)

- Template : Standard Client Task Sequence

![image-26.png](assets/image-26.png)

- Sélectionner l’image Windows importée

![image-27.png](assets/image-27.png)

- Laisser la clé produit vide

![image-28.png](assets/image-28.png)

- Définir un mot de passe pour le compte admin local

![image-29.png](assets/image-29.png)

- Valider le process est terminé

![image-30.png](assets/image-30.png)

## 6️ Lier l’application à la Task Sequence

- Clic droit sur la Task Sequence → Properties

![image-31.png](assets/image-31.png)

- Onglet State Restore
- Section Install Applications
- Cocher : Install a single application
- Choisir VLC

![image-32.png](assets/image-32.png)

![image-33.png](assets/image-33.png)

## 7️ Personnaliser les fichiers de réponse

📁 Dossier : C:\DeploymentShare\Control

### a. CustomSettings.ini

```ini
[Settings]
Priority=Default
[Default]
OSInstall=Y
SkipCapture=YES
UserDataLocation=AUTO
TimeZoneName=Central European Time
KeyboardLocale=040c:0000040c
AdminPassword=<MOT_DE_PASSE>
JoinDomain=<domaine>
DomainAdmin=<domaine>\administrateur
DomainAdminPassword=<MOT_DE_PASSE>
HideShell=YES
ApplyGPOPack=NO
SkipAppsOnUpgrade=NO
SkipAdminPassword=YES
SkipProductKey=YES
SkipComputerName=YES
SkipDomainMembership=YES
SkipUserData=YES
SkipLocaleSelection=YES
SkipTaskSequence=NO
SkipTimeZone=YES
SkipApplications=NO
SkipBitLocker=YES
SkipSummary=YES
SkipFinalSummary=NO
```

### b. Bootstrap.ini

```ini
[Settings]
Priority=Default
[Default]
DeployRoot=\\<SERVEUR>\DeploymentShare$
UserDomain=<domaine>
UserID=administrateur
UserPassword=<MOT_DE_PASSE>
SkipBDDWelcome=YES
```

## 8️ Générer les images personnalisées

### a. Mise à jour du partage MDT

- Clic droit sur MDT Deployment Share → Update Deployment Share

![image-34.png](assets/image-34.png)

- Choisir : Completely regenerate boot images

![image-35.png](assets/image-35.png)

- Voici le message lorsque la procédure s’est terminée correctement

![image-36.png](assets/image-36.png)

### b. Récupérer les images

- Boot : LiteTouchPE_x64.wim → 📁 C:\DeploymentShare\Boot
- Install : install.wim → 📁 C:\DeploymentShare\Operating Systems\installperso

## 9️ Ajouter les images à WDS

### a. Dans la console **WDS**

- Section Images de démarrage :

![image-37.png](assets/image-37.png)

- Ajouter : LiteTouchPE_x64.wim

![image-38.png](assets/image-38.png)

- Ajouter le nom de l’image : Lite Touch Windows PE (x64)

![image-39.png](assets/image-39.png)

- Voici le message lorsque la procédure s’est terminée correctement

![image-40.png](assets/image-40.png)

- Section Images d’installation :

![image-41.png](assets/image-41.png)

- Ajouter : install.wim modifié

![image-42.png](assets/image-42.png)

- Sélectionner l’image souhaitées ici on va prendre seulement Windows10Pro

![image-43.png](assets/image-43.png)

- Voici le message lorsque la procédure s’est terminée correctement

![image-44.png](assets/image-44.png)

## 10 Déployer sur une machine cliente

- Lancer la VM ou le poste → Démarrage PXE (F12 ou boot réseau)

![image-45.png](assets/image-45.png)

- Sélectionner LiteTouch Windows PE (x64)

![image-46.png](assets/image-46.png)

- Suivre l’assistant MDT
- Choisir la Task Sequence : WIN10-CUSTOM

![image-47.png](assets/image-47.png)

- Installation automatisée

![image-48.png](assets/image-48.png)

![image-49.png](assets/image-49.png)

## ✅ Résultat final

- Windows est installé automatiquement

![image-50.png](assets/image-50.png)

- VLC est installé sans intervention

![image-51.png](assets/image-51.png)

- L’ordinateur est directement intégré au domaine <domaine>

![image-52.png](assets/image-52.png)

- Tout est configuré sans interaction manuelle
