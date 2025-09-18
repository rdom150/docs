# Partage Réseau sur Windows Server 2022

## Introduction & Prérequis

La mise en place d'un partage réseau sous Windows Server 2022 permet de partager des fichiers au sein du réseau local via SMB. Ce guide détaille la configuration via le Gestionnaire de serveur.

### Prérequis

- Windows Server 2022 installé et opérationnel
- Accès administratif au Gestionnaire de serveur
- Services de fichiers et de stockage activés
- Comptes utilisateurs prêts
- Un volume disponible pour héberger le partage

---

## Gestionnaire de serveur

Lancez l'application « Gestionnaire de serveur ».

## Services de Fichiers & Stockage

Dans le Gestionnaire de serveur, sélectionnez « Services de fichiers et de stockage ».

![Services Fichiers & Stockage](../../assets/partage-reseau/SHARE1.png)

## Créer nouveau Partage

Sous « Partages », clic droit > « Nouveau Partage ».

![Nouveau partage](../../assets/partage-reseau/SHARE2.png)

### Configuration Rapide de SMB

Choisissez « SMB Rapide ».

![SMB rapide](../../assets/partage-reseau/SHARE3.png)

### Sélectionner le Volume

Sélectionnez le volume à partager.

![Sélection volume](../../assets/partage-reseau/SHARE4.png)

### Nommer le Partage

Donnez un nom descriptif au partage.

![Nom du partage](../../assets/partage-reseau/SHARE5.png)

### Autoriser la Mise en Cache du Partage

Activez l’option de mise en cache si nécessaire.

![Mise en cache](../../assets/partage-reseau/SHARE6.png)

### Configurer les Autorisations

- Cliquez sur « Personnaliser » dans « Autorisations »
- Désactivez l’héritage du compte Admin
- Supprimez les objets hérités et ajoutez les comptes requis

![Autorisations](../../assets/partage-reseau/SHARE7.png)

### Finaliser la Configuration

Cliquez sur « Suivant », vérifiez, puis « Créer » pour finaliser.

---

## Résultat

Félicitations ! Le partage réseau est configuré. Testez l’accès depuis un autre poste.

Liens utiles:
- https://www.it-connect.fr/windows-server-2022-creer-son-premier-partage-de-fichiers-smb/
- https://support.microsoft.com/fr-fr/windows/partage-de-fichiers-sur-un-réseau-dans-windows-b58704b2-f53a-4b82-7bc1-80f9994725bf
