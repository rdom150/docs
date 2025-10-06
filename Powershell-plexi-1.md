<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# avec tous ces document fait moi une doc en .md correct qui rassemvle TOUS avec toutes les expplication possible + les commandes

Voici une documentation complète et modulaire en français au format Markdown, intégrant tous les contenus, explications, scripts et commandes PowerShell pour l'administration Windows, Active Directory et la manipulation avancée avec PowerShell, issus de l'ensemble de tes fichiers BTS SIO. Cette version regroupe les bases, les TP pas à pas, des explications sur les cmdlets, la gestion de l'Active Directory, des exemples de scripts et les usages avancés, tout en structurant le contenu de façon pédagogique et exploitable pour tes travaux pratiques ou révisions.

***

# Administration Windows \& Active Directory avec PowerShell

## Introduction à PowerShell

PowerShell est un langage de script conçu pour l’administration des systèmes Windows, en particulier côté serveur. Il combine une interface de ligne de commande, un langage de script puissant et permet l'automatisation de nombreuses tâches d'administration système.[^1][^2]
PowerShell s'appuie sur le framework .NET et propose des applets de commandes (cmdlets) au format Verbe-Nom (ex : `Get-Process`), permettant d’effectuer des actions précises sur des objets système.

***

## Lancement et utilisation de PowerShell

- **Ouvrir PowerShell** : Menu démarrer > Accessoires > Windows PowerShell (exécuter en tant qu’administrateur).
- **Historique des commandes** : touche `F7` pour rappeler les commandes précédentes.
- **Complétion automatique** : touche `TAB` pour compléter commandes ou chemins.
- **Aide intégrée** :
`Get-Help nom_commande`
`Get-Help Get-ChildItem -Examples`

***

## Structure des commandes (Cmdlets)

- Format : `Verbe-Nom` (ex : `Get-Process`, `Set-ExecutionPolicy`).
- Liste toutes les commandes disponibles :
`Get-Command`
- Chercher aide détaillée :
`Get-Help nom_commande`

***

## Variables et opérateurs

- **Variables** : commencent par `$` (ex : `$maVariable = "Bonjour"`).
- **Opérateurs** :
    - `-eq` (égal), `-ne` (différent), `-gt` (supérieur), `-lt` (inférieur), `-like` (correspondance), etc.

***

## Gestion des scripts

- **Modifier la politique d'exécution** :
Vérifier : `Get-ExecutionPolicy`
Autoriser les scripts locaux non signés :
`Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine`[^3]

***

## Alias de commandes

Certaines commandes possèdent plusieurs alias pour faciliter la transition depuis d’autres environnements (Linux ou CMD). Exemple :

- `Get-ChildItem` ≈ `ls`, `dir`

***

## Filtres et pipelines

Permet de chaîner les commandes (ex : sortir, filtrer, traiter une liste).

- Utiliser le pipe `|` :
`Get-Process | Where-Object {$_.CPU -gt 1000}`
- Exemple :
`Get-ChildItem C:\ | Set-Content fichiers.txt`

***

## Boucles et conditions

- **If/Else** :

```powershell
if ($var -eq 10) { Write-Host "Valeur est 10" }
elseif ($var -eq 5) { Write-Host "Valeur est 5" }
else { Write-Host "Autre valeur" }
```

- **Boucle ForEach** :

```powershell
$nombres = 1..5
foreach ($n in $nombres) { Write-Host $n }
```

- **While/Do** :

```powershell
$i = 0
while ($i -lt 10) { Write-Host $i; $i++ }
```


***

## Exemples de Scripts Powershell (Bases)

### 1. Rechercher un fichier dans un dossier

```powershell
param(
  [string]$NomFichier,
  [string]$Dossier
)
Get-ChildItem -Path $Dossier -Recurse -ErrorAction SilentlyContinue | Where-Object { $_.Name -eq $NomFichier } | ForEach-Object { Write-Host "Le fichier $NomFichier est dans $($_.DirectoryName)" }
```


### 2. Calculer la taille des fichiers d’un dossier

```powershell
param([string]$Dossier)
Get-ChildItem -Path $Dossier -Recurse -Force | Where-Object { -not $_.PsIsContainer } | Measure-Object -Property Length -Sum | ForEach-Object {
  $totalMB = [math]::Round($_.Sum / 1MB, 2)
  Write-Host -ForegroundColor Yellow "Le dossier $Dossier contient $totalMB MB"
}
```


***

## Gestion de l’Active Directory en PowerShell

### Installation de l’AD PowerShell module

```powershell
Add-WindowsFeature RSAT-AD-PowerShell
Import-Module ActiveDirectory
```


### Commandes de base

- Créer une OU (Organisation Unit) :

```powershell
New-ADOrganizationalUnit -Name "Employes" -Path "dc=monentreprise,dc=local"
```

- Créer un utilisateur :

```powershell
New-ADUser -Name "Paul Bismuth" -GivenName "Paul" -Surname "Bismuth" -SamAccountName "pbismuth" -UserPrincipalName "pbismuth@monentreprise.com" -AccountPassword (Read-Host -AsSecureString "Entrer mot de passe") -PassThru | Enable-ADAccount
```

- Créer un groupe :

```powershell
New-ADGroup -Name "Politique" -GroupScope Global -Path "ou=Employes,dc=monentreprise,dc=local"
```

- Ajouter un utilisateur à un groupe :

```powershell
Add-ADGroupMember -Identity "Politique" -Members "pbismuth"
```

- Désactiver/activer un compte :

```powershell
Disable-ADAccount -Identity "pbismuth"
Enable-ADAccount -Identity "pbismuth"
```

- Supprimer un utilisateur :

```powershell
Remove-ADUser -Identity "pbismuth"
```

- Lister tous les utilisateurs :

```powershell
Get-ADUser -Filter * | Format-List
```


***

### Import de plusieurs utilisateurs via CSV

Script pour ajouter automatiquement des utilisateurs à partir d’un fichier CSV (par exemple, `utilisateurs.csv`) :

```powershell
Import-Csv -Path "utilisateurs.csv" | ForEach-Object {
  $securePassword = ConvertTo-SecureString $_.Password -AsPlainText -Force
  New-ADUser -Name $_.Name -GivenName $_.GivenName -Surname $_.Surname -SamAccountName $_.Login -UserPrincipalName $_.Mail `
    -AccountPassword $securePassword -Path $_.OU -Enabled $true
}
```


***

### Gestion avancée AD (Sélection de cmdlets utiles)

| Cmdlet | Description |
| :-- | :-- |
| Get-ADUser | Récupère un ou plusieurs utilisateurs |
| New-ADUser | Crée un nouvel utilisateur |
| Set-ADUser | Modifie un utilisateur |
| Remove-ADUser | Supprime un utilisateur |
| Get-ADGroup | Récupère un ou plusieurs groupes AD |
| Add-ADGroupMember | Ajoute des membres à un groupe |
| New-ADOrganizationalUnit | Crée une OU |
| Remove-ADOrganizationalUnit | Supprime une OU |

Pour lister toutes les cmdlets du module AD :

```powershell
Get-Command -Module ActiveDirectory
```


***

## Administration d’un Serveur Windows Core avec PowerShell

### Installation et configuration de Windows Server Core

- Paramétrer adresse IP, DNS, nom d’hôte :

```powershell
New-NetIPAddress -IPAddress "172.16.0.1" -PrefixLength 24 -InterfaceIndex (Get-NetAdapter).ifIndex -DefaultGateway "172.16.0.254"
Set-DnsClientServerAddress -InterfaceIndex (Get-NetAdapter).ifIndex -ServerAddresses "127.0.0.1","8.8.8.8"
Rename-Computer -NewName "SRV-WIN-CORE1" -Force
Restart-Computer
```

- Installer le rôle ADDS et promouvoir le serveur contrôleur de domaine :

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "monentreprise.local" -DomainNetbiosName "MONENTREPRISE" -CreateDnsDelegation:$false -DatabasePath "C:\Windows\NTDS" -DomainMode "Default" -ForestMode "Default" -InstallDns:$true -LogPath "C:\Windows\NTDS" -SysvolPath "C:\Windows\SYSVOL" -Force:$true
Restart-Computer
```


***

### Installation et configuration d’un serveur DHCP via PowerShell

- Installer le rôle DHCP :

```powershell
Install-WindowsFeature -Name DHCP -IncludeManagementTools
```

- Créer une étendue DHCP :

```powershell
Add-DhcpServerv4Scope -Name "ScopeSodecaf" -StartRange 172.16.0.150 -EndRange 172.16.0.200 -SubnetMask 255.255.255.0 -State Active
Set-DhcpServerv4OptionValue -ScopeId 172.16.0.0 -DnsServer 172.16.0.1,8.8.8.8 -Router 172.16.0.254
```


Pour vérifier les rôles installés :

```powershell
Get-WindowsFeature | Where-Object {$_.Installed -eq $true}
```


***

## Exemples de scripts avancés pour TP

- **Colorer la sortie selon la fonction dans un CSV**

```powershell
Import-Csv -Path "utilisateurs.csv" | ForEach-Object {
    $color = switch ($_.function) {
        "informatique" {"Cyan"}
        "direction conseil" {"Red"}
        "comptable" {"Yellow"}
        "social" {"Blue"}
        "juridique" {"White"}
    }
    Write-Host "$($_.firstname) $($_.lastname) $($_.phone1)" -ForegroundColor $color
}
```

- **Créer des dossiers par agence et employés**

```powershell
Import-Csv -Path "utilisateurs.csv" | ForEach-Object {
    $trigramme = ($_.firstname.Substring(0,1) + $_.lastname.Substring(0,1) + $_.lastname.Substring($_.lastname.Length-1,1)).ToUpper()
    $dirAgence = "C:\Agences\$($_.agence)"
    $dirUser = "$dirAgence\fic$trigramme"
    if (!(Test-Path $dirAgence)) { New-Item -Path $dirAgence -ItemType Directory }
    if (!(Test-Path $dirUser)) { New-Item -Path $dirUser -ItemType Directory }
}
```


***

## Trucs \& Astuces PowerShell

- **Continuer sur plusieurs lignes** :
Utiliser le backtick ````` en fin de ligne.
- **Afficher la date** :
`Get-Date -Format "dd-MM-yyyy"`
- **Afficher le contenu d’un fichier** :
`Get-Content fichier.txt`
- **Pipe (`|`)** : Envoie la sortie d'une commande en entrée d'une autre.
- **Aide rapide** :
`help Get-Process`

***

## Pour aller plus loin

- Utilisation des modules externes (NTFSSECURITY pour gestion avancée des droits NTFS)
- Administration à distance (WinRM, WAC - Windows Admin Center)
- Scripts d’import, gestion fine des mots de passe et gestion réseau

***

**Ressources officielles** :

- Microsoft Docs PowerShell
- IT-Connect (exemples, tutoriaux)

***

Ce document couvre l'ensemble des attentes d’un administrateur réseau ou d’un étudiant souhaitant maîtriser PowerShell pour l’administration systèmes, la gestion Active Directory et l'automatisation des TPs BTS SIO.[^2][^1][^4][^5][^6][^3]

<div align="center">⁂</div>

[^1]: B2-Act4-TP2-pws-bases-1.pdf

[^2]: B2-Act4-cours-Powershell.pdf

[^3]: B2-Act4-TP3-pws-niveau-2-1.pdf

[^4]: B2-Act4-TP5-Powershell-AD-1.pdf

[^5]: B2-Act4-TP5-Powershell-AD.pdf

[^6]: B2-Act4-TP4-Windows-Core-et-PowerShell.pdf

