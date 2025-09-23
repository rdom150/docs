# ⚡ Guide pratique : PowerShell – Bases & Scripts

Ce document regroupe les **commandes PowerShell essentielles** vues en TP, avec des explications détaillées et des exemples pratiques.

---

## 🔹 1. Découverte des cmdlets

Une cmdlet suit la forme **Verbe-Nom** (ex : `Get-Command`).  
Les principaux verbes utilisés :  
- **Get** : obtenir  
- **Set** : définir/modifier  
- **Remove** : supprimer  
- **New** : créer  
- **Write** : afficher  

```powershell
Get-Command -CommandType cmdlet
```
👉 Liste toutes les cmdlets disponibles dans PowerShell.  

```powershell
Get-Command write-*
```
👉 Affiche toutes les cmdlets dont le nom commence par `write`.  

```powershell
Get-Command *-Object
```
👉 Affiche toutes les cmdlets qui manipulent des objets.  

---

## 🔹 2. Alias de commandes

PowerShell intègre des **alias** pour faciliter la transition depuis MS-DOS ou Linux.

```powershell
Get-ChildItem C:\Users
ls C:\Users
dir C:\Users
```

👉 Ces trois commandes affichent le contenu d’un dossier.  
- `Get-ChildItem` = commande officielle (verbe-nom).  
- `ls` = alias style Linux.  
- `dir` = alias style MS-DOS.  

---

## 🔹 3. Navigation & gestion des fichiers

```powershell
cd C:\Windows
pwd
ls -R
ls -Force C:\
```

- `cd` : change de répertoire.  
- `pwd` : affiche le répertoire courant.  
- `ls -R` : liste récursive (parcourt tous les sous-dossiers).  
- `ls -Force` : inclut les fichiers cachés et systèmes.  

---

## 🔹 4. Variables

Les variables commencent par `$` et leur type est déterminé automatiquement.

```powershell
$maVariable = "Bonjour le monde"
$maVariable | Get-Member
$maVariable.ToUpper()
$maVariable.Length
```

- `$maVariable` : crée une variable contenant une chaîne.  
- `Get-Member` : liste les propriétés et méthodes de l’objet.  
- `.ToUpper()` : met en majuscules.  
- `.Length` : donne le nombre de caractères.  

---

## 🔹 5. Caractères génériques

```powershell
ls C:\Windows\*.ini
ls C:\Windows\*p*.*
```

- `?` : remplace un seul caractère.  
- `*` : remplace une suite de caractères.  

Exemple :  
- `ls C:\Windows\*.ini` → liste tous les fichiers `.ini`.  
- `ls C:\Windows\*p*.*` → liste tous les fichiers contenant la lettre `p`.  

---

## 🔹 6. Aides à la saisie

```powershell
cd ~
```
👉 `~` correspond au répertoire personnel de l’utilisateur.  

```powershell
get-ch<TAB>
```
👉 Auto-complétion : `TAB` complète le nom de la commande ou du fichier.  

---

## 🔹 7. Commandes multi-lignes

PowerShell permet de couper une commande longue avec le **backtick** (`` ` ``).

```powershell
Get-EventLog -LogName System `
  -Newest 25 `
  -EntryType Error,Warning
```

- Le backtick (AltGr + 7) permet de continuer la commande sur plusieurs lignes.  
- `-Newest 25` : affiche les 25 derniers événements.  
- `-EntryType Error,Warning` : filtre uniquement erreurs et avertissements.  

---

## 🔹 8. Manipulation de fichiers (exercice)

```powershell
cd ~
md tp_pws
cd tp_pws
Get-Date > essai.txt
"une première ligne de renseignements" >> essai.txt
"suivi par une seconde ligne" >> essai.txt
cat essai.txt
$ligne = cat essai.txt
Write-Host $ligne
```

- `md` : crée un dossier.  
- `>` : redirige la sortie vers un fichier (écrase le fichier).  
- `>>` : ajoute à la fin du fichier.  
- `cat` : lit le contenu du fichier.  
- `Write-Host` : affiche dans la console.  

👉 Cet exemple crée un fichier texte, y écrit deux lignes, puis les relit et les affiche.  

---

## 🔹 9. Pipes et filtres

Le **pipe** (`|`) envoie le résultat d’une commande vers une autre.

```powershell
Get-ChildItem C:\Windows | Set-Content monWindows.txt
```
👉 Sauvegarde la liste des fichiers Windows dans `monWindows.txt`.  

```powershell
Get-Process | Out-File -FilePath process.txt
```
👉 Sauvegarde la liste des processus dans un fichier texte.  

```powershell
Get-Service | Where {$_.Status -eq "Stopped"}
```
👉 Affiche uniquement les services arrêtés.  

Explication :  
- `Get-Service` → liste tous les services.  
- `Where-Object` (`where` ou `?`) → filtre les résultats.  
- `$_` → représente chaque objet de la liste (ici un service).  

---

## 🔹 10. Interaction avec l’utilisateur

```powershell
$invite = "Donnez votre nom :"
$user = Read-Host $invite
Write-Output ("Bonjour : " + $user)
```

👉 Affiche une invite, récupère le nom de l’utilisateur, puis l’affiche.  

---

## 🔹 11. Gestion des dates

```powershell
$date = Get-Date -f "dd-MM-yyyy"
Write-Output ("La date du jour est : " + $date)
Write-Output ("Le mois numérique est : " + (Get-Date).Month)
```

👉 Récupère la date actuelle et affiche jour + mois.  

Propriétés utiles :  
- `Day`, `DayOfWeek`, `Month`, `Year`, `Hour`, `Minute`, `Second`.  

Méthodes utiles :  
- `AddDays(nb)` → ajoute des jours.  
- `AddMonths(nb)` → ajoute des mois.  

Exemple :

```powershell
Get-EventLog -LogName System -EntryType Error -After (Get-Date).AddDays(-2)
```
👉 Liste les erreurs système survenues dans les 2 derniers jours.  

---

# ✅ Conclusion

Avec ces commandes PowerShell, tu maîtrises les bases :  
- Navigation et manipulation de fichiers.  
- Variables et chaînes de caractères.  
- Pipes et filtres pour traiter les résultats.  
- Interaction utilisateur et gestion des dates.  

Ce pense-bête peut servir de base pour rédiger des scripts plus avancés.  

---
