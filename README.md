# Daloradius+openvpn+pihole+unbound en  debian 11 para mikrotik
Se realiza la instalacion de Daloradius ,en conjunto con Pihole y unbound :shipit:

- [x] Daloradius
- [x] Openvpn
- [x] Pihole
- [x] Unbound

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
nano authorized_keys
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
apt install software-properties-common dirmngr -y
apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://mirror.rackspace.com/mariadb/repo/10.5/debian bullseye main'
apt update
```

```
apt install mariadb-server mysqltuner -y
systemctl start mysql.service
mysql_secure_installation
# enter
# Disallow root login remotely? [Y/n] n
# password usada aqui es 84Uniq@
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
mysql -u root -p radius < etc/freeradius/3.0/mods-config/sql/main/mysql/schema.sql
ln -s etc/freeradius/3.0/mods-available/sql etc/freeradius/3.0/mods-enabled/
nano etc/freeradius/3.0/mods-enabled/sql
apt install -y git
```

```
git clone https://github.com/wirisp/Daloradius-Wireguard-Pihole-unbound.git dapiun
\mv /root/dapiun/sql /etc/freeradius/3.0/mods-available/sql
nano etc/freeradius/3.0/mods-enabled/sql
```

```
chgrp -h freerad etc/freeradius/3.0/mods-available/sql
chown -R freerad:freerad etc/freeradius/3.0/mods-enabled/sql
```

```
systemctl restart freeradius
apt -y install wget zip unzip
```

```
git clone https://github.com/wirisp/daloradius.git
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
\mv /root/dapiun/radiusd.conf etc/freeradius/3.0/radiusd.conf
nano /etc/freeradius/3.0/radiusd.conf
```

```
\mv /root/dapiun/index.php /var/www/html/index.php
```

```
\mv /root/dapiun/default etc/freeradius/3.0/sites-enabled/default
nano etc/freeradius/3.0/sites-enabled/default
```

```
\mv /root/dapiun/sqlcounter etc/freeradius/3.0/mods-available/sqlcounter
#nano etc/freeradius/3.0/mods-available/sqlcounter
```

```
\mv /root/dapiun/access_period.conf /etc/freeradius/3.0/mods-config/sql/counter/mysql/access_period.conf
#nano /etc/freeradius/3.0/mods-config/sql/counter/mysql/access_period.conf
```

```
\mv /root/dapiun/quotalimit.conf /etc/freeradius/3.0/mods-config/sql/counter/mysql/quotalimit.conf
#nano /etc/freeradius/3.0/mods-config/sql/counter/mysql/quotalimit.conf
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
cd dapiun
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

### backup db
```
mysqldump -p -u root radius > dbname.sql
```
### Restaurar db
```
mysql -p -u root radius < dbname.sql
```

## Instalacion de Openvpn para mikrotik en Servidor debian 11
- Primero descargamos el script a nuestro sistema , ene ste caso, debian 11.
```
wget https://raw.githubusercontent.com/volstr/openvpn-install-routeros/main/openvpn-install-routeros.sh -O openvpn-install-routeros.sh
#wget https://raw.githubusercontent.com/wirisp/openvpn-pihole-unbound/main/openvpn-install-routeros.sh -O openvpn-install-routeros.sh
```
- Despues ejecutamos el script descargado
```
sudo bash ./openvpn-install-routeros.sh
```
> si te pregunta el tipo de conexion , selecciona tcp

- Despues cambiamos los dns con

```
nano /etc/openvpn/server/server.conf
```

_Comentamos las lineas siguientes y en su lugar colocamos una nueva con la ip que usaremos de DNS_
```
#Stop using Google DNS for our OpenVPN
#push "dhcp-option DNS 8.8.8.8"
#push "dhcp-option DNS 8.8.4.4"
```
- Colocamos esta nueva

```
push "dhcp-option DNS 10.8.0.1"
```
- Reiniciamos el servicio de openvpn con

```
systemctl start openvpn-server@server.service
systemctl status openvpn-server@server.service
```


### Instalacion de Pihole en debian 11
```
systemctl stop wg-quick@wg0.service
wg-quick down wg0
```
_Pihole no esta soportado en debian 11 por lo que mostrara error primero_
```
curl -sSL https://install.pi-hole.net | bash
```
_Despues lanzamos el siguiente comando desactivando la comprobacion de sistema_
```
#curl -sSL https://install.pi-hole.net | sudo PIHOLE_SKIP_OS_CHECK=true bash
curl -sSL https://install.pi-hole.net | PIHOLE_SKIP_OS_CHECK=true bash
```

En la configuracion, seleccionar la interfaz que fue creada al instalar openvpn, la cual deveria ser `tun0` o algo parecido.

_Cambiamos el password con_
```
PIHOLE_SKIP_OS_CHECK=true sudo -E pihole -a -p
```

_Iniciamos los servicios con_
```
systemctl start wg-quick@wg0.service
systemctl status wg-quick@wg0.service
```

## Instalacion y configuracion de unbound
Instalamos unbound con el siguiente comando, no antes de darle permisos
```
wget https://raw.githubusercontent.com/wirisp/openvpn-pihole-unbound/main/unbound.sh -O unbound.sh
```

```
chmod +x *.sh
./unbound.sh 
```
Despues editamos el archivo

```
> /etc/pihole/setupVars.conf 
```

```
echo "PIHOLE_INTERFACE=tun0
QUERY_LOGGING=true
INSTALL_WEB_SERVER=true
INSTALL_WEB_INTERFACE=true
LIGHTTPD_ENABLED=true
CACHE_SIZE=10000
DNS_FQDN_REQUIRED=true
DNS_BOGUS_PRIV=true
DNSMASQ_LISTENING=single
WEBPASSWORD=a31c87c18e9ff2eca7edb3aa0f7ee8ec24e92157a6f55d873115fd4084c37b0c
BLOCKING_ENABLED=true
PIHOLE_DNS_1=127.0.0.1#5335
PIHOLE_DNS_2=127.0.0.1#5335
DNSSEC=false
REV_SERVER=false" >> /etc/pihole/setupVars.conf 
```

- Activamos con

```
systemctl enable pihole-FTL
```
- Cambio de password con

```
pihole -a -p
```

_Reiniciamos_
```
reboot
```


## Conexion de openvpn a mikrotik
- Subir los archivos del cliente correspondientes, al Administrador de archivos, en este caso el cliente se llama Mk17, por lo que se suben `Mk17.crt` y `Mk17.key`
<img width="454" alt="image" src="https://user-images.githubusercontent.com/13319563/222213812-80b61638-2fc8-4ee0-b79e-902e7316d32d.png">
- Despues en la terminal los importamos


```
certificate import file-name=Mk17.crt
certificate import file-name=Mk17.key
```

- Creamos el perfil que usaremos

```
ppp profile add name=OVPN-client change-tcp-mss=yes only-one=yes use-encryption=yes use-mpls=no use-compression=no
```
- Creamos la inteface ppp para ovpn
```
interface ovpn-client add name=ovpn-client connect-to=xxx.xxx.xxx.xxx port=1194 mode=ip user="openvpn" password="" profile=OVPN-client certificate=Mk17.crt_0 auth=sha1 cipher=blowfish128 add-default-route=yes
```
- Cambia los Dns en 
```
/ip dns
set allow-remote-requests=yes servers=10.8.0.1
```

y tambien en 

```
/ip dhcp-server network
```
>Listo ya tenemos configurado y corriendo Openvpn con pihole y unbound, podemos administrar desde `IP/admin`

