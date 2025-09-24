
Voici une explication de la réplication en temps réel Master/Slave MySQL avec les commandes formatées en markdown pour faciliter la copie, basée sur le contenu du lien que vous avez fourni.

***

## Explication de la réplication Master/Slave MySQL

La réplication Master/Slave permet de synchroniser en temps réel une base de données "Slave" avec une base "Master". Le "Master" reçoit toutes les écritures (INSERT, UPDATE, DELETE) et écrit ces modifications dans un fichier binaire (binlog). Ce fichier est lu par le "Slave" qui applique les mêmes changements pour avoir une copie à jour de la base de données.

Cette architecture sert à assurer une haute disponibilité et une tolérance aux pannes, en permettant au "Slave" de prendre le relais si le "Master" est défaillant.

***

## Étapes et commandes pour la mise en place

### 1. Préparation des serveurs MySQL Master et Slave

Mettre à jour les paquets et installer MySQL sur les deux serveurs :

```bash
apt-get update
apt-get install mysql-server
```


***

### 2. Configuration du serveur Master

Modifier le fichier de configuration MySQL du Master (`/etc/mysql/my.cnf`) pour lui affecter une adresse IP d'écoute et un identifiant unique :

```
[mysqld]
server-id = 1
bind-address = 192.168.1.20
log_bin = /var/log/mysql/mysql-bin.log
```

Créer les fichiers de log et attribuer les bons droits :

```bash
touch /var/log/mysql/mysql-bin.log
touch /var/log/mysql/mysql-error.log
chown mysql:mysql /var/log/mysql/mysql-bin.log /var/log/mysql/mysql-error.log
```

Redémarrer MySQL pour prendre en compte les changements :

```bash
service mysql restart
```

Créer l'utilisateur dédié à la réplication qui sera utilisé par le Slave pour se connecter :

```sql
CREATE USER 'slave'@'192.168.1.21' IDENTIFIED BY '<password>';
GRANT REPLICATION SLAVE ON *.* TO 'slave'@'192.168.1.21';
FLUSH PRIVILEGES;
```


***

### 3. Création d'une base de test sur le Master

Connectez-vous au MySQL Master et créez une base et une table de test :

```sql
CREATE DATABASE test1;
USE test1;
CREATE TABLE T1 (ID INT NOT NULL) ENGINE=InnoDB CHARACTER SET latin1 COLLATE latin1_bin;
INSERT INTO T1 (ID) VALUES (1);
```


***

### 4. Verrouiller les tables du Master pour la sauvegarde

Afin d'assurer une copie cohérente, verrouiller les tables en lecture seule :

```sql
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
```

Notez le nom du fichier binaire (`File`) et la position (`Position`) affichés par la commande `SHOW MASTER STATUS`.

***

### 5. Sauvegarder la base de données du Master et la transférer vers le Slave

Sur le Master, faire un dump SQL de la base test1 :

```bash
mysqldump -u root -p --databases test1 > test1.sql
```

Transférer ce fichier sur le Slave :

```bash
scp test1.sql user@192.168.1.21:/root
```


***

### 6. Configuration du serveur Slave

Sur le serveur Slave, modifier `/etc/mysql/my.cnf` pour ajouter un identifiant unique différent du Master et activer les logs :

```
[mysqld]
server-id = 2
relay-log = /var/log/mysql/mysql-relay-bin.log
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = test1
```

Redémarrer MySQL sur le Slave :

```bash
service mysql restart
```


***

### 7. Importer la base sur le Slave

Se connecter au Slave et importer la base de données :

```bash
mysql -u root -p < /root/test1.sql
```


***

### 8. Configurer la réplication sur le Slave

Se connecter à MySQL sur le Slave et arrêter tout processus de réplication en cours :

```sql
STOP SLAVE;
```

Configurer la connexion au Master avec les informations du fichier binaire et de la position relevées :

```sql
CHANGE MASTER TO 
  MASTER_HOST='192.168.1.20',
  MASTER_USER='slave',
  MASTER_PASSWORD='<password>',
  MASTER_LOG_FILE='<nom_fichier_binaire>',
  MASTER_LOG_POS=<position>;
```

Démarrer la réplication :

```sql
START SLAVE;
```

Vérifier que la réplication fonctionne bien :

```sql
SHOW SLAVE STATUS\G;
```

Les valeurs `Slave_IO_Running` et `Slave_SQL_Running` doivent être à `Yes`.

***

### 9. Test de la réplication

Sur le Master, insérer une nouvelle valeur :

```sql
USE test1;
INSERT INTO T1 (ID) VALUES (2);
```

Sur le Slave, vérifier la présence de cette nouvelle valeur :

```sql
USE test1;
SELECT * FROM T1;
```


***

### 10. Simulation de panne et remise en ordre

Si le Master tombe, on peut écrire sur le Slave. Pour remettre la réplication :

- Sur le Slave, faire un dump de la base :

```bash
mysqldump -u root -p --databases test1 > test1.sql
```

- Arrêter la réplication du Slave :

```sql
STOP SLAVE;
```

- Transférer la sauvegarde vers le Master et la restaurer :

```bash
scp test1.sql user@192.168.1.20:/root
service mysql start
mysql -u root -p < /root/test1.sql
```

- Sur le Master, récupérer le nouveau fichier binaire et la position :

```sql
SHOW MASTER STATUS;
```

- Sur le Slave, mettre à jour la configuration avec ces nouvelles valeurs, puis démarrer la réplication :

```sql
CHANGE MASTER TO 
  MASTER_LOG_FILE='<nouveau_fichier_binaire>',
  MASTER_LOG_POS=<nouvelle_position>;
START SLAVE;
```


***

Ces étapes et commandes couvrent la mise en place basique, le test et la remise en service d'une réplication MySQL Master/Slave.

Si besoin, les configurations peuvent évoluer suivant la version de MySQL, notamment à partir de la 5.5, mais le principe général reste similaire.

***

N'hésitez pas à utiliser les commandes en markdown ci-dessus pour copier-coller dans vos terminaux ou scripts.
Cette synthèse est conforme au tutoriel détaillé sur [https://www.it-connect.fr/replication-en-temps-reel-masterslave-mysql%EF%BB%BF/](https://www.it-connect.fr/replication-en-temps-reel-masterslave-mysql%EF%BB%BF/).[^1_1]

```
<div style="text-align: center">⁂</div>
```

[^1_1]: https://www.it-connect.fr/replication-en-temps-reel-masterslave-mysql﻿/


---

# je veut que tu me fasse le fichier compler pour que je puisse juste lenregistré

Voici un exemple complet de fichier de configuration MySQL pour un serveur Master et un serveur Slave, prêt à être enregistré et utilisé, basé sur le tutoriel fourni.

***

## Fichier de configuration complet pour le Master (`/etc/mysql/my.cnf`)

```ini
[mysqld]
server-id = 1
bind-address = 192.168.1.20
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = test1
```


***

## Fichier de configuration complet pour le Slave (`/etc/mysql/my.cnf`)

```ini
[mysqld]
server-id = 2
relay-log = /var/log/mysql/mysql-relay-bin.log
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = test1
```


***

Ces fichiers sont les configurations minimales nécessaires pour mettre en place la réplication Master/Slave pour la base de données `test1`.

Après avoir enregistré ces fichiers, n'oubliez pas de créer les fichiers de logs et d'attribuer les bons droits sur le Master :

```bash
touch /var/log/mysql/mysql-bin.log
touch /var/log/mysql/mysql-error.log
chown mysql:mysql /var/log/mysql/mysql-bin.log /var/log/mysql/mysql-error.log
service mysql restart
```

Et redémarrer MySQL sur le Slave aussi :

```bash
service mysql restart
```


***

Si une version plus complète ou avec d'autres paramètres est souhaitée (comme pour la sécurité ou la performance), merci de le préciser.
Ce sont les fichiers essentiels pour que le système de réplication fonctionne selon le tutoriel partagé.

---

# fait moi pareil pour lui [https://www.it-connect.fr/comment-mettre-en-place-mariadb-galera-cluster-sur-debian-11/](https://www.it-connect.fr/comment-mettre-en-place-mariadb-galera-cluster-sur-debian-11/)

Voici une explication synthétique pour mettre en place un cluster MariaDB Galera sur Debian 11, accompagnée d’un fichier de configuration complet prêt à être enregistré, basée sur le tutoriel du lien fourni.

***

## Présentation

MariaDB Galera Cluster permet d’avoir un cluster de bases de données multi-maîtres avec réplication synchrone en temps réel sur plusieurs nœuds. Il garantit la haute disponibilité et la tolérance aux pannes. Le cluster doit contenir un nombre impair de nœuds (3, 5, 7, etc.) pour éviter le phénomène de split-brain, et fonctionne uniquement avec les moteurs InnoDB ou XtraDB.

***

## Préparation et installation

Sur **chacun des nœuds** (ex. IPs 192.168.100.51, 192.168.100.52, 192.168.100.53) :

Mettre à jour et installer MariaDB avec Galera :

```bash
sudo apt-get update
sudo apt-get install mariadb-server galera-4
```


***

## Configuration de MariaDB Galera Cluster

Modifier le fichier de configuration `/etc/mysql/mariadb.conf.d/60-galera.cnf` sur **chaque nœud** pour inclure cette configuration (adapter les IP et le nom du nœud) :

```ini
[galera]
# Activation du provider wsrep
wsrep_on = ON
wsrep_provider = /usr/lib/galera/libgalera_smm.so

# Nom du cluster (identique sur tous les nœuds)
wsrep_cluster_name = "Galera_Cluster_IT-Connect"

# Adresse des nœuds du cluster
wsrep_cluster_address = gcomm://192.168.100.51,192.168.100.52,192.168.100.53

# Format du binlog pour la réplication
binlog_format = ROW

# Moteur de stockage par défaut
default_storage_engine = InnoDB

# Mode auto-incrémentation et forçage de clé primaire
innodb_autoinc_lock_mode = 2
innodb_force_primary_key = 1

# Paramètres réseau
bind-address = 0.0.0.0

# Identification unique du nœud (adapter le nom et l'adresse IP de chaque nœud)
wsrep_node_name = "SRV-DEB-1"
wsrep_node_address = "192.168.100.51"

# Fichier de log des erreurs
log_error = /var/log/mysql/error-galera.log
```

Pour les autres nœuds, remplacer `wsrep_node_name` et `wsrep_node_address` par leurs propres identifiants et IP respectifs (`SRV-DEB-2`, `192.168.100.52`, etc.).

***

## Démarrage du cluster

1. Sur le premier nœud (ex. SRV-DEB-1), stopper MariaDB :
```bash
sudo systemctl stop mariadb
```

2. Initialiser le cluster :
```bash
sudo galera_new_cluster
```

3. Vérifier que le cluster est actif :
```bash
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size';"
```

Le résultat doit être `1` au début.

4. Sur les autres nœuds, démarrer MariaDB normalement :
```bash
sudo systemctl start mariadb
```

Ils rejoindront automatiquement le cluster.

***

## Vérification de l'état du cluster

Sur chaque nœud, vérifier que le nœud est synchronisé :

```bash
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_local_state_comment';"
```

Le résultat attendu est `Synced`.

***

## Remarques supplémentaires

- Assurez-vous que le firewall laisse passer les ports 3306, 4444, 4567, 4568.
- Les bases doivent utiliser InnoDB ou XtraDB, avec une clé primaire sur chaque table.
- Pour que le service web puisse basculer sur un autre nœud du cluster en cas de panne, pensez à utiliser une IP virtuelle (VIP) avec Keepalived, HAProxy ou un équilibrage DNS multiple.
- Pour autoriser les connexions distantes, dans le fichier `/etc/mysql/mariadb.conf.d/50-server.cnf`, modifiez `bind-address` à `0.0.0.0`.
- Donnez les droits nécessaires à l’utilisateur MariaDB qui sera utilisé par l’application :

```sql
GRANT ALL PRIVILEGES ON `wordpress`.* TO 'utilisateur-bdd-wp'@'<ip_serveur_web>' IDENTIFIED BY 'Mot-de-Passe' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```


***

Cette configuration et procédure vous permettent d’établir un cluster MariaDB Galera fonctionnel sur Debian 11.

***

Si besoin, voici à copier pour le fichier `/etc/mysql/mariadb.conf.d/60-galera.cnf` d’un nœud (adapter `wsrep_node_name` et `wsrep_node_address`) :

```ini
[galera]
wsrep_on = ON
wsrep_provider = /usr/lib/galera/libgalera_smm.so
wsrep_cluster_name = "Galera_Cluster_IT-Connect"
wsrep_cluster_address = gcomm://192.168.100.51,192.168.100.52,192.168.100.53
binlog_format = ROW
default_storage_engine = InnoDB
innodb_autoinc_lock_mode = 2
innodb_force_primary_key = 1
bind-address = 0.0.0.0
wsrep_node_name = "SRV-DEB-1"
wsrep_node_address = "192.168.100.51"
log_error = /var/log/mysql/error-galera.log
```


***

