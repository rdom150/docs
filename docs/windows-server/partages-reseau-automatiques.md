# Configuration Automatisée des Partages Réseau et des Dossiers Utilisateurs

## Introduction & Prérequis

Ce guide vous montre comment automatiser le montage de partages réseau pour les utilisateurs de l'Active Directory (AD) sur un serveur Windows. En outre, nous verrons comment créer automatiquement un dossier personnel pour chaque utilisateur.

### Prérequis

- Un serveur Windows fonctionnel
- Un utilisateur et une machine cliente dans le domaine Active Directory
- Une connaissance de base des partages réseau sous Windows Server

---

## Scripts pour les Partages Réseau

### Création des Partages

Tout d'abord, créez vos partages réseau avec les autorisations appropriées. Assurez-vous qu'ils sont accessibles et configurés selon vos besoins.

![Partages réseau](../../assets/partages-reseau-automatiques/networkshareauto01.png)

### Écriture du Script de Montage

Ensuite, créez un script de montage pour chaque partage. Écrivez les scripts à l'emplacement approprié (par exemple un partage \NETLOGON ou un dossier centralisé de scripts).

![Emplacement des scripts](../../assets/partages-reseau-automatiques/networkshareauto02.png)

Exemple de contenu pour un script de connexion:

```bat
net use Z: \\serveur\partage1
net use Y: \\serveur\partage2
```

![Exemple de script](../../assets/partages-reseau-automatiques/networkshareauto03.png)

### Attribution du Script dans l'AD

Dans l'Active Directory, accédez au profil de l'utilisateur et spécifiez le chemin complet du script de montage dans la section d'ouverture de session.

![Attribution script AD](../../assets/partages-reseau-automatiques/networkshareauto04.png)

### Résultat

> La Première partie est fonctionnelle

![Résultat script](../../assets/partages-reseau-automatiques/networkshareauto05.png)

---

## Configuration du Dossier Personnel

### Configuration des Autorisations

Configurez les autorisations du dossier utilisateur pour garantir la sécurité et l'accessibilité appropriées.

![Autorisations dossier](../../assets/partages-reseau-automatiques/networkshareauto06.png)

### Attribution de la Lettre de Lecteur dans l'AD

Dans l'Active Directory, configurez le dossier personnel de l'utilisateur en lui attribuant une lettre de lecteur pour un accès facile.

![Lettre de lecteur AD](../../assets/partages-reseau-automatiques/networkshareauto07.png)

### Résultat

![Résultat dossier personnel](../../assets/partages-reseau-automatiques/networkshareauto08.png)

---

Référence: https://docs.ldurand-it.fr/fr/Services/Windows-Server/networkshareauto
