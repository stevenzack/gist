MySQL

- [Initialization](#initialization)
- [Trigger](#trigger)
- [Procedure](#procedure)
- [Event](#event)
- [FullText Search](#fulltext-search)
- [SSL Encrypted Connection](#ssl-encrypted-connection)
- [Binlog Replication](#binlog-replication)
- [GTID Replication](#gtid-replication)
- [XtraBackup](#xtrabackup)
- [Partitioning](#partitioning)


# MariaDB

```sh
sudo apt install mariadb-server
systemctl start mariadb
sudo mariadb –u root
```

# MySQL

Download MySQL from [Oracle](https://dev.mysql.com/downloads/mysql/)

```sh
sudo gdebi mysql-apt-config_0.8.30-1_all.deb
sudo apt update
sudo apt install mysql-server
mysql -u root -p
```

Find your first temp password for root account
```sh
grep 'temporary password' /var/log/mysqld.log
mysql -u root -p
>alter user 'root'@'localhost' identified by 'yourpassword';
```

# Initialization

```sql
>create user forum@'%' identified by '12345678';
>grant all privileges on *.* to forum@'%' with grant option;
>exit;
```

# Trigger

```sql
mysql> delimiter //
mysql> CREATE TRIGGER on_account_type_changes BEFORE UPDATE ON users
       FOR EACH ROW
       BEGIN
           IF NEW.account_type != OLD.account_type THEN
               update messages set account_type=NEW.account_type where user_id=OLD.id;
           ELSEIF NEW.amount > 100 THEN
               SET NEW.amount = 100;
           END IF;
       END;//
mysql> delimiter ;
```

# Procedure

```sql
drop procedure if exists insert_many_rows;

delimiter //

create procedure insert_many_rows (IN loops int)
begin
	declare v1 int;
	set v1 = loops;
	while v1 > 0 do
		insert into test_table(age) values(v1);
		set v1 = v1 - 1;
	end while;
end;
//

delimiter ;
```
Use procedure
```sql
call insert_many_rows(10);
```

# Event

```sql
create event insert_users_event on schedule every 2 minute
do
insert into users(name) values('asd');
```

# FullText Search

/etc/mysql/mysql.conf.d/mysqld.cnf

```cnf
[mysqld]
ngram_token_size = 1
```

```sql
create table posts(id int auto_increment primary key,title varchar(100),body varchar(200),fulltext (title,body) with parser ngram);

insert into posts(title,body) values('MySQL教程','学习MySQL快速，简单和有趣全文');
insert into posts(title,body) values('MySQL全文搜索','MySQL提供了具有许多好的功能的内置全文搜索');

select * from posts where match (title,body) against ('全');
```


# SSL Encrypted Connection

Configure MySQL
```sh
vim /etc/mysql/mysql.conf.d/mysqld.cnf
```
Add few configuration, this will generate certificate files in `/var/lib/mysql` directory.
```cnf
[mysqld]
ssl_ca=ca.pem
ssl_cert=server-cert.pem
ssl_key=server-key.pem
require_secure_transport=ON
```


Client connection
```go
// verifyPeerCertFunc returns a function that verifies the peer certificate is
// in the cert pool.
func verifyPeerCertFunc(pool *x509.CertPool) func([][]byte, [][]*x509.Certificate) error {
	return func(rawCerts [][]byte, _ [][]*x509.Certificate) error {
		if len(rawCerts) == 0 {
			return errors.New("no certificates available to verify")
		}

		cert, err := x509.ParseCertificate(rawCerts[0])
		if err != nil {
			return err
		}

		opts := x509.VerifyOptions{Roots: pool}
		if _, err = cert.Verify(opts); err != nil {
			return err
		}
		return nil
	}
}
func main() {
	rootCertPool := x509.NewCertPool()
	pem, err := os.ReadFile("/home/asd/ca.pem")
	if err != nil {
		log.Fatal(err)
	}
	if ok := rootCertPool.AppendCertsFromPEM(pem); !ok {
		log.Fatal("Failed to append PEM.")
	}
	cert, err := tls.LoadX509KeyPair("/home/asd/client-cert.pem", "/home/asd/client-key.pem")
	if err != nil {
		log.Fatal("failed to load key pair")
	}
	omysql.RegisterTLSConfig("custom", &tls.Config{
		RootCAs:               rootCertPool,
		Certificates:          []tls.Certificate{cert},
		InsecureSkipVerify:    true,
		VerifyPeerCertificate: verifyPeerCertFunc(rootCertPool),
	})
	// sql.Open("mysql","username:password@localhost:3306/dbname?parseTime=true&tls=custom")
}
```

# Binlog Replication

On source mysql instance:

`sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf`

```conf
[mysqld]
log-bin = mysql-bin
server-id = 1
```

`sudo systemctl restart mysql`

In first session on source
```sql
>flush tables with read lock;
>show binary log status\G;
```
Remember the binlog file name, and the position of binlog. Don't exit the session, or the lock will be released.

---
On slave mysql instance

`sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf`

```conf
[mysqld]
server-id=2
```

`sudo systemctl restart mysql`

```sql
>CHANGE REPLICATION SOURCE TO
    ->     SOURCE_HOST='192.168.10.12',
    ->     SOURCE_USER='replication_user_name',
    ->     SOURCE_PASSWORD='replication_password',
    ->     SOURCE_LOG_FILE='mysql-bin.000001',
    ->     SOURCE_LOG_POS=365;
```

```sql
>start replica;
```

# GTID replication

On source instance:

`>set @@GLOBAL.read_only = ON;`

`sudo vim /etc/mysql/mysql.conf.d/mysqld.conf`

```conf
[mysqld]
server-id	= 1
gtid_mode	= ON
enforce-gtid-consistency = ON
```


`sudo systemctl restart mysql`

On replica instance:


```sql
>stop replica;
>set @@GLOBAL.super_read_only = ON;
```

`sudo vim /etc/mysql/mysql.conf.d/mysqld.conf`

```conf
[mysqld]
server-id	= 1
gtid_mode	= ON
enforce-gtid-consistency = ON
super-read-only = ON
```

`sudo systemctl restart mysql`

```sql
>change replication source to
source_host='source_hostname',
source_user='username',
source_password='password',
source_auto_position=1;
>start replica;
```

# XtraBackup

Download [XtraBackup 8.3.0.deb](https://www.percona.com/downloads)

```sh
dpkg -i percona-xtrabackup-83_8.3.0-1-1.bookworm_amd64.deb 
apt -f install
```

Download [mysql-server_8.3.0-1debian12_amd64.deb-bundle.tar](https://downloads.mysql.com/archives/community/)

```sh
mkdir bundle/
cd bundle/
tar xvf ../mysql-server_8.3.0-1debian12_amd64.deb-bundle.tar
```

Install mysql 8.3.0
```sh
apt install libaio1 -y
dpkg-preconfigure mysql-community-server_*.deb
dpkg -i mysql-{common,community-client-plugins,community-client-core,community-client,client,community-server-core,community-server,server}_*.deb
apt -f install
```

Backup
```sh
mkdir backup/
sudo xtrabackup --backup --user=root --password=mypassword --target-dir=./backup
sudo 7zr a backup.7z backup/
```

Restore
```sh
7zr x backup.7z
xtrabackup --prepare --target-dir=./backup/
systemctl stop mysql
mv /var/lib/mysql /var/lib/mysql2
cp -r ./backup/ /var/lib/mysql
chown mysql -R /var/lib/mysql
chgrp mysql -R /var/lib/mysql
systemctl start mysql
```

# Partitioning

```sql
create table orders(
	id int not null,
	created date not null,
	primary key(created,id)
) partition by range (YEAR(created)) (
	partition p_old values less than (2019),
	partition p_2019 values less than (2020),
	partition p_max values less than MAXVALUE
);
```

Query

```sql
select * from orders partition (p2019);
select * from orders where created between '2019-01-01' and '2019-12-31' and id=1;
```
