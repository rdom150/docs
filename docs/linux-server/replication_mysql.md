# 📌 Guide pratique : Réplication MySQL Maître ↔ Esclave

Ce document regroupe les **commandes essentielles** et un **pense-bête dépannage** pour configurer et maintenir une réplication MySQL.

---

## 🔹 1. Configuration du serveur Maître

Modifier le fichier `/etc/mysql/mariadb.conf.d/50-server.cnf` :

```ini
[mysqld]
server-id = 100
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = nom_de_la_base
```

- `server-id` : identifiant unique (différent sur chaque serveur).  
- `log_bin` : active les logs binaires nécessaires à la réplication.  
- `binlog_do_db` : indique la base à répliquer.  

---

## 🔹 2. Configuration du serveur Esclave

Modifier le fichier `/etc/mysql/mariadb.conf.d/50-server.cnf` :

```ini
[mysqld]
server-id = 104
master-retry-count = 20
replicate-do-db = nom_de_la_base
```

- `server-id` : identifiant unique, différent du maître.  
- `replicate-do-db` : base à répliquer.  
- `master-retry-count` : nombre de tentatives de reconnexion.  

---

## 🔹 3. Sur le Maître – Préparation

Verrouiller les tables pour figer l’état de la base :

```sql
FLUSH TABLES WITH READ LOCK;
```

Obtenir le **nom du fichier binlog** et la **position** :

```sql
SHOW MASTER STATUS;
```

➡️ Noter les valeurs `File` et `Position`. Elles seront utilisées sur l’esclave.  

---

## 🔹 4. Sur l’Esclave – Arrêter la réplication

```sql
STOP SLAVE;
```

---

## 🔹 5. Sur l’Esclave – Indiquer les infos du Maître

Adapter avec les bonnes valeurs (`MASTER_LOG_FILE` et `MASTER_LOG_POS`) :

```sql
CHANGE MASTER TO
  MASTER_HOST='172.16.0.10',
  MASTER_USER='replicateur',
  MASTER_PASSWORD='motdepasse',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=3921;
```

---

## 🔹 6. Sur l’Esclave – Démarrer la réplication

```sql
START SLAVE;
```

---

## 🔹 7. Sur le Maître – Débloquer les écritures

```sql
UNLOCK TABLES;
```

---

## 🔹 8. Vérifier la réplication

Sur l’esclave :

```sql
SHOW SLAVE STATUS \G;
```

Vérifier que :  
- `Slave_IO_Running: Yes`  
- `Slave_SQL_Running: Yes`  

✅ Si les deux sont à **Yes**, la réplication fonctionne.  

---

# 🛠️ Dépannage & Maintenance

## ⚠️ Erreur fréquente : données désynchronisées

1. Sauvegarder la base du maître (depuis l’esclave, après avoir stoppé la réplication) :

```bash
mysqldump -h 172.16.0.10 -u root -p nom_de_la_base > sauvegarde.sql
```

2. Restaurer la sauvegarde sur l’esclave :

```bash
mysql -u root -p nom_de_la_base < sauvegarde.sql
```

3. Relancer la réplication (sur l’esclave) :

```sql
STOP SLAVE;
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=48044;
START SLAVE;
```

---

## 🔄 Purger les anciens binlogs sur le maître

Lister les fichiers binlog :

```sql
SHOW MASTER LOGS;
```

Supprimer les anciens binlogs :

```sql
PURGE MASTER LOGS TO 'mysql-bin.000027';
```

⚠️ Ne jamais supprimer manuellement les fichiers binlog avec `rm`, sinon MySQL perd le suivi.  

---

# ✅ Conclusion

Avec ces commandes, tu peux :  
- Configurer la réplication **maître ↔ esclave**  
- Vérifier son état  
- Réparer une désynchronisation  
- Gérer les binlogs

---
