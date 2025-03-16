## Kompose2Helm

Use of kompose to convert docker-compose file to helm charts
Very simple and fast but the output adds annotation to the helm charts.

The purpose of this repo was to test conversion to helm charts that will work with Kubernetes and Rancher Fleet .

Output does not match the same folder structure that Fleet expects.

### Issues
Noticed the servas app doesn't spin up properly  - 500 Server Errors.

The following is troubleshooting to resolve the errors.

docker exec -it servas mysql -u root -p -h servas-app-db-1 --skip-ssl

Without the `--skip-ssl`command, there's the following error:
````
mysql: Deprecated program name. It will be removed in a future release, use '/usr/bin/mariadb' instead
ERROR 2026 (HY000): TLS/SSL error: SSL is required, but the server does not support it
````
SELECT user, host, authentication_string FROM mysql.user WHERE user IN ('root', 'servas_db_user');
### Checking Connection from Servas App (Frontend) to MariaDB (Backend)
`docker exec -it servas nc -zv servas-app-db-1 3306`
````
servas-app-db-1 (172.20.0.2:3306) open
````

````
/app # which nc
/usr/bin/nc
/app # nc -zv servas-app-db-1 3306
servas-app-db-1 (172.20.0.2:3306) open

````

```
/app # mysql -u servas_db_user -p<> 
mysql: Deprecated program name. It will be removed in a future release, use '/usr/bin/mariadb' instead
ERROR 2002 (HY000): Can't connect to local server through socket '/run/mysqld/mysqld.sock' (2)
/app # mysql -u servas_db_user -p<secret> -h servas-app-db-1
mysql: Deprecated program name. It will be removed in a future release, use '/usr/bin/mariadb' instead
ERROR 2026 (HY000): TLS/SSL error: SSL is required, but the server does not support it
/app # mysql -u servas_db_user -p<secret> -h servas-app-db-1 --skip-ssl
mysql: Deprecated program name. It will be removed in a future release, use '/usr/bin/mariadb' instead
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 5
Server version: 10.7.3-MariaDB-1:10.7.3+maria~focal mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SHOW TABLES;
ERROR 1046 (3D000): No database selected
MariaDB [(none)]> USE mysql;
ERROR 1044 (42000): Access denied for user 'servas_db_user'@'%' to database 'mysql'
MariaDB [(none)]> USE servas_db;
Database changed
MariaDB [servas_db]> SHOW TABLES;
Empty set (0.000 sec)

MariaDB [servas_db]> 

```

### Check as root and still no tables in servas_db
````
MariaDB [servas_db]> exit
Bye
/app # mysql -u root -p<secret> -h servas-app-db-1 --skip-ssl
mysql: Deprecated program name. It will be removed in a future release, use '/usr/bin/mariadb' instead
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6
Server version: 10.7.3-MariaDB-1:10.7.3+maria~focal mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> USE servas_db;
Database changed
MariaDB [servas_db]> SHOW TABLES;
Empty set (0.001 sec)

MariaDB [servas_db]> 

````