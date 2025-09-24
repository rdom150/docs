


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

