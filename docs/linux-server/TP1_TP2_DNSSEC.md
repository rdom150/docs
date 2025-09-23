# TP – Mise en place et sécurisation d’un DNS (BIND9 + DNSSEC)

Ce document regroupe **l’ensemble des explications et commandes** vues dans la conversation :  
-  Mise en place d’un serveur DNS BIND9 (primaire + secondaire)  
-  Sécurisation avec DNSSEC  

---

# 🔹 Mise en place d’un serveur DNS BIND9

## 1. Préparer l’environnement

- Importer la VM **Debian 12** et la renommer :

```bash
hostnamectl set-hostname srv-dns1
```

- Configurer l’IP fixe (exemple : `172.16.0.3/24`) :

```bash
nano /etc/network/interfaces
```

Exemple :
```
auto ens33
iface ens33 inet static
    address 172.16.0.3
    netmask 255.255.255.0
    gateway 172.16.0.1
    dns-nameservers 8.8.8.8
```

Puis :
```bash
systemctl restart networking
```

- Mise à jour et installation :

```bash
apt update && apt upgrade -y
apt install bind9 dnsutils -y
```

- Cloner la VM → renommer en **srv-dns2** avec IP `172.16.0.4`.

---

## 2. Configurer le serveur DNS primaire (srv-dns1)

### Fichier `/etc/bind/named.conf.local`

```conf
zone "sodecaf.fr" {
    type master;
    file "/var/cache/bind/db.sodecaf.fr";
    allow-transfer { 172.16.0.4; };
};

zone "0.16.172.in-addr.arpa" {
    type master;
    file "/var/cache/bind/db.172.16.0.rev";
    allow-transfer { 172.16.0.4; };
};
```

### Fichier de zone directe `/var/cache/bind/db.sodecaf.fr`

```zone
$TTL 86400
@   IN  SOA ns1.sodecaf.fr. admin.sodecaf.fr. (
            2025092201 ; Serial
            86400      ; Refresh
            21600      ; Retry
            3600000    ; Expire
            3600 )     ; Negative Cache TTL

    IN  NS   ns1.sodecaf.fr.
    IN  NS   ns2.sodecaf.fr.

ns1 IN  A    172.16.0.3
ns2 IN  A    172.16.0.4
www IN  A    172.16.0.10
```

### Fichier de zone inverse `/var/cache/bind/db.172.16.0.rev`

```zone
$TTL 86400
@   IN  SOA ns1.sodecaf.fr. admin.sodecaf.fr. (
            2025092201 ; Serial
            86400
            21600
            3600000
            3600 )
    IN  NS   ns1.sodecaf.fr.
    IN  NS   ns2.sodecaf.fr.

3   IN  PTR  ns1.sodecaf.fr.
4   IN  PTR  ns2.sodecaf.fr.
10  IN  PTR  www.sodecaf.fr.
```

### Exemple complet avec plusieurs serveurs (vu en cours)

```zone
$TTL 86400
@   IN  SOA srv-dns1.sodecaf.fr. hostmaster.sodecaf.fr. (
            2025092201 ; serial
            86400      ; refresh
            21600      ; retry
            3600000    ; expire
            3600 )     ; negative cache TTL

@   IN  NS  srv-dns1.sodecaf.fr.
@   IN  NS  srv-dns2.sodecaf.fr.

3   IN  PTR srv-dns1.sodecaf.fr.
4   IN  PTR srv-dns2.sodecaf.fr.
10  IN  PTR srv-web1.sodecaf.fr.
11  IN  PTR srv-web2.sodecaf.fr.
12  IN  PTR www.sodecaf.fr.
```

---

## 3. Configurer le serveur secondaire (srv-dns2)

### Fichier `/etc/bind/named.conf.local`

```conf
zone "sodecaf.fr" {
    type slave;
    file "/var/cache/bind/slave/db.sodecaf.fr";
    masters { 172.16.0.3; };
};

zone "0.16.172.in-addr.arpa" {
    type slave;
    file "/var/cache/bind/slave/db.172.16.0.rev";
    masters { 172.16.0.3; };
};
```

Créer le répertoire :

```bash
mkdir -p /var/cache/bind/slave
chown bind:bind /var/cache/bind/slave
```

---

## 4. Vérifications et tests

- Vérifier la syntaxe :

```bash
named-checkconf
named-checkzone sodecaf.fr /var/cache/bind/db.sodecaf.fr
named-checkzone 0.16.172.in-addr.arpa /var/cache/bind/db.172.16.0.rev
```

- Redémarrer :

```bash
systemctl restart bind9
systemctl status bind9
```

- Tester depuis un **client Linux** :

```bash
dig @172.16.0.3 www.sodecaf.fr
dig -x 172.16.0.10
```

- Tester depuis un **client Windows** :

```cmd
nslookup www.sodecaf.fr 172.16.0.3
```

---

## 5. Redirection des requêtes externes

Dans `/etc/bind/named.conf.options` :

```conf
options {
    directory "/var/cache/bind";
    forwarders { 172.17.0.1; };
    allow-query { any; };
};
```

Puis :

```bash
systemctl restart bind9
```

---

#  Activer DNSSEC sur BIND9

## 1. Préparer l’environnement

```bash
cd /etc/bind
mkdir keys
```

Activer DNSSEC dans `/etc/bind/named.conf.options` :

```conf
options {
    directory "/var/cache/bind";
    dnssec-enable yes;
    dnssec-validation auto;
    listen-on-v6 { any; };
};
```

---

## 2. Générer les clés

### ZSK (Zone Signing Key)

```bash
cd /etc/bind/keys
dnssec-keygen -a RSASHA256 -b 1024 -n zone sodecaf.fr
```

### KSK (Key Signing Key)

```bash
dnssec-keygen -a RSASHA256 -b 2048 -f KSK -n zone sodecaf.fr
```

---

## 3. Modifier le fichier de zone

Ajouter à la fin de `/var/cache/bind/db.sodecaf.fr` :

```zone
$INCLUDE "/etc/bind/keys/Ksodecaf.fr.+008+<ZSK_ID>.key"
$INCLUDE "/etc/bind/keys/Ksodecaf.fr.+008+<KSK_ID>.key"
```

---

## 4. Signer la zone

```bash
cd /etc/bind/keys
dnssec-signzone -o sodecaf.fr -k Ksodecaf.fr.+008+<KSK_ID> /var/cache/bind/db.sodecaf.fr Ksodecaf.fr.+008+<ZSK_ID>
```

👉 Génère : `/var/cache/bind/db.sodecaf.fr.signed`

---

## 5. Utiliser la zone signée

Modifier `/etc/bind/named.conf.local` :

```diff
-zone "sodecaf.fr" {
-    type master;
-    file "/var/cache/bind/db.sodecaf.fr";
-    allow-transfer { 172.16.0.4; };
-};
+zone "sodecaf.fr" {
+    type master;
+    file "/var/cache/bind/db.sodecaf.fr.signed";
+    allow-transfer { 172.16.0.4; };
+};
```

---

## 6. Vérification et redémarrage

```bash
named-checkzone sodecaf.fr /var/cache/bind/db.sodecaf.fr.signed
systemctl restart bind9
```

---

## 7. Tests DNSSEC

```bash
dig +dnssec @172.16.0.3 sodecaf.fr SOA
dig dnskey sodecaf.fr
```

👉 Les réponses doivent contenir des enregistrements **RRSIG** (signatures).

---

# ✅ Résumé

- **TP1** : installation, configuration d’un DNS maître + esclave avec résolution directe et inverse.  
- **TP2** : ajout de **DNSSEC** → génération clés ZSK/KSK, signature de la zone, validation avec `dig +dnssec`.  

Ton serveur DNS `sodecaf.fr` est maintenant **fonctionnel et sécurisé**.
