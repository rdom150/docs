# Active Directory Windows Server 2022


> **[Active Directory]**  
> L'Active Directory est un annuaire LDAP pour les systèmes d'exploitation Windows, le tout étant créé par Microsoft. Cet annuaire contient différents objets, de différents types (utilisateurs, ordinateurs, etc.), l'objectif étant de centraliser deux fonctionnalités essentielles : l'identification et l'authentification au sein d'un système d'information.

**ADillustration**

### Prérequis

Pour suivre ce guide dans de bonnes conditions assurez vous d'avoir à disposition un Windows Server 2022 avec le mot de passe administrateur.  
Ayez aussi une machine Windows 10 / 11 pour effectuer de futurs test que l'on ne verra pas dans ce guide.

> **Explication détaillée** : Dans ce guide nous partons d'un fresh install de Windows Server 2022 donc quelques parties seront dédiés à la configuration du Serveur.
> 
> * Naviguez dans la barre de droite pour vous déplacez à l'endroit qui vous intéresse.

## Première connexion

### Mot De Passe Administrateur

Après avoir terminé l'installation de Windows Server 2022 en suivant ce guide vous allez devoir définir un mot de passe robuste pour votre serveur.  
Pour des questions de sécurités évidentes ce mot de passe ne doit être connu & accessible uniquement aux Administrateurs Systèmes & Réseaux de votre entreprise.

![Mot de passe administrateur](../../assets/assets-AD/ImageAD1.png)

> **[Sécurité 🔐]**  
> Utilisez un gestionnaire de mots de passe (ex: KeePass) pour générer et stocker vos mots de passe.
> 
> **Explication détaillée** :
> 1. 12+ caractères avec majuscules, minuscules, chiffres et symboles
> 2. Rotation tous les 90 jours
> 3. Accès limité aux administrateurs

### Déverrouillage (VM PVE)

Pour accèder à votre Window Server vous devez d'abord le déverrouillez en utilisant les touches `CTRL` + `ALT` + `SUPPR`.  
Cela ne pose pas réellement de problème lorsque vous êtes sur un serveur Physique mais lorsque vous utiliser un VM vous devez utiliser les raccourcis rédéfinis par noVNC.

* Cliquez sur le Chevron à votre gauche pour déployer la barre latérale
* Cliquez sur l'icone de Touche A
* Cliquez sur l'icone des trois touches tout en bas.

![Déverrouillage VM PVE](../../assets/assets-AD/ImageAD2.png)

Désormais vous avez accès à l'écran de connexion.  
Saisissez vos Identifiants et patientez.

> **[CHARGEMENT ...]**  
> La première connexion peut durer quelques minutes selon les ressources.
> 
> **Explication détaillée** :
> 1. **Temps de connexion** : La première connexion peut prendre 2-5 minutes selon les ressources allouées
> 2. **Configuration automatique** : Windows Server configure automatiquement les services au premier démarrage
> 3. **Patience requise** : Évitez d'interrompre le processus de démarrage initial

### Gestionnaire de Serveur

Le Gestionnaire de serveur est une console de gestion disponible dans Windows Server, qui aide les professionnels de l'informatique à provisionner et à gérer des serveurs Windows locaux ou distants directement depuis leur Bureau, sans avoir besoin d'accéder physiquement aux serveurs ni d'activer des connexions RDP (Remote Desktop Protocol) pour chaque serveur.

Le Gestionnaire de Serveur se lancera automatiquement à chaque démarrage de Windows Server.

![Gestionnaire de serveur](../../assets/assets-AD/ImageAD3.png)

> **Explication détaillée** :
> 1. Console centralisée
> 2. Démarrage automatique
> 3. Installation des rôles et surveillance

## Configuration du Serveur

### Adressage IP

Pour ce guide je suis dans un environnement de test et donc je ne suis pas soumis à à un adressage IP prédéfinie.  
Je vais alors procéder de cette manière :

* IP Windows Serveur : _192.168.50.1 /24_
* Passerelle Par défaut : _192.168.50.254_
* DNS ce sera notre Windows Serveur alors on met _127.0.0.1_ soit lui-même

![Configuration adressage IP](../../assets/assets-AD/ImageAD4.png)

> **Explication détaillée** :
> 1. IP statique recommandée
> 2. DNS local : 127.0.0.1
> 3. Configurez via Paramètres réseau Windows

### Renommer le Serveur

On va désormais nommer notre serveur en remplaçant celui par défaut.  
Pour cela rendez vous sur le Gestionnaire De Serveur :

* Cliquez sur Serveur Local puis sur son nom.
* Dans Propriétés Systèmes clique sur modifier puis sur autres
* Renseigner le domain et le nom , le nom complet s'affichera après

![Renommer le serveur](../../assets/assets-AD/ImageAD5.png)

> **[NOMMER SON SERVEUR]**  
> Pour nommer votre serveur soyez assez simple et explicite pour pouvoir l'identifier facilement.  
> Dans mon cas j'ai nommer mon serveur : _WINSRV01_
> 
> **Explication détaillée** :
> 1. Convention claire (ex: WINSRV01, DC01)
> 2. Lettres, chiffres, tirets uniquement
> 3. Redémarrage requis après changement

Après avoir nommer votre serveur vous allez être dans l'obligation de redémarrez votre serveur

### Démarrage des services.

Après avoir redémarrez votre serveur et vous être reconnecter vous arrivez une nouvelle fois sur le Gestionnaire De Serveur.  
Ne le fermez pas nous allons vérifier que tous les services soient bien démarrés.  
Pour cela :

* Cliquez sur la partie services de votre serveur local
* Un fenêtre s'ouvre
* Faites un clique droit sur les services et selectionnez démarrez les services.

![Démarrage des services](../../assets/assets-AD/ImageAD6.png)

Vous êtes désormais passé dans le vert.

> **[Passage au Vert]**  
> Si c'est encore en rouge, démarrez les services manuellement.
> 
> **Explication détaillée** :
> 1. Vérifiez les services critiques
> 2. Passez au vert = OK
> 3. En cas d'erreur, consultez l'observateur d'événements

## Installation d'Active Directory

L'étape cruciale de l'installation d'Active Directory est maintenant devant vous. Active Directory (AD) joue un rôle central dans la gestion des utilisateurs, des groupes, et des ressources dans un environnement Windows. Suivez ces instructions pour intégrer votre serveur dans le monde d'Active Directory.

### Ajouter le Rôle AD

Dans le gestionnaire de serveur en haut à droite cliquez sur Gérer

* Sélectionnez ensuite "Ajouter des Rôles et Fonctionnalités"
* Vous êtes désormais dans l'assistant d'ajout de rôle et de fonctionnalités .
* Faites Suivant et sélectionnez votre serveur Local sur lequel vous souhaitez ajoutez l'Active Directory
* Ensuite selectionnez le rôle "AD DS" puis ajoutez les fonctionnalités
* Après vérifiez tout dans la fenêtre de confirmation puis installer

![Ajouter le rôle AD DS](../../assets/assets-AD/ImageAD7.png)

> **Explication détaillée** :
> 1. Ajoutez le rôle AD DS
> 2. Acceptez les fonctionnalités proposées
> 3. Validez et installez

![Sélection des fonctionnalités AD](../../assets/assets-AD/ImageAD8.png)

### Promouvoir en Contrôleur de Domaine

Désormais nous sommes à une étape primordiale qui est de promouvoir notre serveur en contrôleur de domaine.  
Pour cela à la fin de l'installation de Active Directory ne fermez pas la fenêtre de l'assistant et cliquez sur le lien bleu.

![Promouvoir en contrôleur de domaine](../../assets/assets-AD/ImageAD9.png)

On va maintenant configurer le déploiement d'Active Directory.

* Choisissez votre nom de domaine qui correspond à ce que vous avez choisi ici
* Pas de Délégation DNS
* Laisser les chemins d'accès par défaut

![Configuration du déploiement AD](../../assets/assets-AD/ImageAD10.png)

> **Explication détaillée** :
> 1. Choisissez le nom de domaine interne
> 2. Laissez les chemins par défaut
> 3. Définissez un mot de passe DSRM fort

![Configuration DNS](../../assets/assets-AD/ImageAD11.png)

![Options de configuration](../../assets/assets-AD/ImageAD12.png)

> **Explication détaillée** :
> 1. **Configuration DNS** : Le serveur DNS sera installé automatiquement
> 2. **Options de configuration** : Vérifiez toutes les options avant la promotion
> 3. **Vérification des prérequis** : L'assistant vérifie automatiquement les prérequis
> 4. **Installation** : La promotion peut prendre 10-20 minutes selon les performances

## Conclusion

> **[Bien Joué ! ✔️]**  
> Voilà vous avez désormais réussi à installer votre Active Directory sur Windows Serveur 2022.

Explorez les fonctionnalités d'AD pour administrer votre domaine. 🚀

> **Explication détaillée** :
> 1. **Post-installation** : Vérifiez que le domaine est opérationnel
> 2. **Outils d'administration** : Utilisez l'outil "Utilisateurs et ordinateurs Active Directory"
> 3. **Premiers objets** : Créez vos premiers utilisateurs et groupes
> 4. **Sauvegarde** : Planifiez des sauvegardes régulières de votre contrôleur de domaine
> 5. **Sécurité** : Configurez les stratégies de groupe et la sécurité

---

## Prochaines étapes

1. **Configuration des utilisateurs** : Créez vos premiers comptes utilisateurs
2. **Stratégies de groupe** : Configurez les GPO pour sécuriser votre domaine
3. **Sauvegarde** : Mettez en place une stratégie de sauvegarde
4. **Monitoring** : Surveillez la santé de votre contrôleur de domaine
5. **Tests** : Testez l'authentification depuis des postes clients


