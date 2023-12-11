# Daloradius debian 11 para mikrotik
Se realiza la instalacion de Daloradius ,en conjunto con Pihole y unbound :shipit:

- [x] Daloradius

## Preparacion del vps 
- Activar ipv6 en contabo
```
enable_ipv6
nano /etc/ssh/sshd_config
```

```
Port 6813 
PermitRootLogin yes
PasswordAuthentication yes
```

```
passwd root
service ssh restart
systemctl restart sshd
mkdir .ssh
nano /root/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys && chmod 700 ~/.ssh/
apt update
```

```
apt -y install software-properties-common gnupg2 dirmngr
#apt-get install wget curl net-tools gamin
apt -y upgrade
reboot
```

```
apt install software-properties-common dirmngr -y git
apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://mirror.rackspace.com/mariadb/repo/10.5/debian bullseye main'
#add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://mirror.rackspace.com/mariadb/repo/10.5/ubuntu focal main'
apt update
```

```
apt install mariadb-server mysqltuner -y
systemctl start mysql.service
mysql_secure_installation
# enter
# Disallow root login remotely? [Y/n] n
# password usada aqui es 84Uniq@ ,utiliza el tuyo
```

```
mysql -u root -p
CREATE DATABASE radius;
GRANT ALL ON radius.* TO radius@localhost IDENTIFIED BY "84Uniq@";
FLUSH PRIVILEGES;
quit;
```

```
apt -y install apache2
apt -y install php libapache2-mod-php php-{gd,common,mail,mail-mime,mysql,pear,mbstring,xml,curl}
apt -y install freeradius freeradius-mysql freeradius-utils
systemctl enable --now freeradius.service
mysql -u root -p radius < /etc/freeradius/3.0/mods-config/sql/main/mysql/schema.sql
ln -s /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/
#nano /etc/freeradius/3.0/mods-enabled/sql
apt install -y git
```

```
git clone https://github.com/wirisp/Daloradius-Wireguard-Pihole-unbound.git dapiun
\mv /root/dapiun/sql /etc/freeradius/3.0/mods-available/sql
nano /etc/freeradius/3.0/mods-enabled/sql
#Cambia el password 84Uniq@
```

```
chgrp -h freerad /etc/freeradius/3.0/mods-available/sql
chown -R freerad:freerad /etc/freeradius/3.0/mods-enabled/sql
```

```
systemctl restart freeradius
apt -y install wget zip unzip
```

```
git clone https://github.com/wirisp/daloradius-almalinux8.git daloradius
cd daloradius
```

```
mysql -u root -p radius < contrib/db/fr2-mysql-daloradius-and-freeradius.sql 
```

```
mysql -u root -p radius < contrib/db/mysql-daloradius.sql
```

```
cd ..
sudo mv daloradius /var/www/html/
```

```
\mv /root/dapiun/daloradius.conf.php /var/www/html/daloradius/library/daloradius.conf.php
chown -R www-data:www-data /var/www/html/daloradius/
chmod 664 /var/www/html/daloradius/library/daloradius.conf.php
nano /var/www/html/daloradius/library/daloradius.conf.php
#Cambia el password en CONFIG_DB_PASS por el tuyo
```

```
systemctl restart freeradius.service apache2
```

```
timedatectl set-timezone America/Mexico_City
```

```
pear install DB
pear install MDB2
pear channel-update pear.php.net
```


En estos archivos que moveremos, podemos ver que hemos efectuado algunos cambios, hay una leyenda en ellos que dice 
> wirisp Cambiar aqui
```
\mv /root/dapiun/radiusd.conf /etc/freeradius/3.0/radiusd.conf
nano /etc/freeradius/3.0/radiusd.conf
```

```
#Falta el index
\mv /root/dapiun/index.php /var/www/html/index.php
```

```
\mv /root/dapiun/default /etc/freeradius/3.0/sites-enabled/default
nano /etc/freeradius/3.0/sites-enabled/default
```

```
\mv /root/dapiun/sqlcounter /etc/freeradius/3.0/mods-available/sqlcounter
#nano /etc/freeradius/3.0/mods-available/sqlcounter
```

```
\mv /root/dapiun/access_period.conf /etc/freeradius/3.0/mods-config/sql/counter/mysql/access_period.conf
#nano /etc/freeradius/3.0/mods-config/sql/counter/mysql/access_period.conf
```

```
\mv /root/dapiun/quotalimit.conf /etc/freeradius/3.0/mods-config/sql/counter/mysql/quotalimit.conf
#nano /etc/freeradius/3.0/mods-config/sql/counter/mysql/quotalimit.conf
```

```
wget https://raw.githubusercontent.com/wirisp/Daloradius-debian/main/eap -O eap
\mv eap /etc/freeradius/3.0/mods-available/eap
# Activada la opcion para usar NAS-identifier como atributo y limitar a un solo NAS al usuario
```

> Aqui hay cambios que no se hicieron post-auth
```
\mv /root/dapiun/queries.conf /etc/freeradius/3.0/mods-config/sql/main/mysql/queries.conf
nano /etc/freeradius/3.0/mods-config/sql/main/mysql/queries.conf
```

_Cambiamos case sensitive=yes a no_
```
\mv /root/dapiun/radutmp /etc/freeradius/3.0/mods-enabled/radutmp
```
```
#No existe falta base.sql
wget https://raw.githubusercontent.com/wirisp/daloup/main/dbname.sql -O /root/dapiun/base.sql
mysql -p -u root radius < /root/dapiun/base.sql
```

```
systemctl status apache2
systemctl stop freeradius
systemctl daemon-reload
systemctl start freeradius
systemctl status freeradius
```

```
systemctl restart freeradius
```

- Error en daloradius ,serivicio Mysql muestra = disabled
```
nano -l /var/www/html/daloradius/library/exten-radius_server_info.php
```
_Buscamos y Cambiamos Mysql por Mariadb_
```
<td class='summaryKey'> MariaDB </td> <td class='summaryValue'><span class='sleft'><?php echo check_service("mariadb");  ?></span> </td>
```

```
chmod 777  /var/log/syslog
chmod 777 /var/log/freeradius
```
### Iniciar sesion
WEB: IP/daloradius
Usuario: administrator
Pass: 84Uniq@
Pass2 si el primero no funciona: radius

### backup db
```
mysqldump -p -u root radius > dbname.sql
```
### Restaurar db
```
mysql -p -u root radius < dbname.sql
```
## Respaldo carpeta html completa
```
cd /var/www
tar -zcvf html.tar.gz html
```

- Descomprimir con
```
cd /var/www/html
tar -xf html.tar.gz
```

## Instalacion de Openvpn para mikrotik en Servidor debian 11
- La instalacion siguiente continua aqui
```
https://github.com/wirisp/openvpn-pihole-unbound
```

>Listo ya tenemos configurado y corriendo Openvpn con pihole y unbound, podemos administrar desde `IP/admin`

