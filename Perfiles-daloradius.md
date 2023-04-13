## Ejemplo perfil 2 horas pausadas
```
MariaDB [radius]> SELECT * FROM radgroupreply WHERE groupname = '2hPausada';
+----+-----------+-----------------------+----+---------------------------------------------+
| id | groupname | attribute             | op | value                                       |
+----+-----------+-----------------------+----+---------------------------------------------+
| 29 | 2hPausada | Mikrotik-Rate-Limit   | := | 512K/2M 1M/3M 384K/1500K 16/12 8 256K/1000K |
| 30 | 2hPausada | Acct-Interim-Interval | := | 60                                          |
+----+-----------+-----------------------+----+---------------------------------------------+
2 rows in set (0.001 sec)

MariaDB [radius]> SELECT * FROM radgroupcheck WHERE groupname = '2hPausada';
+----+-----------+------------------+----+-------+
| id | groupname | attribute        | op | value |
+----+-----------+------------------+----+-------+
|  5 | 2hPausada | Max-All-Session  | := | 7200  |
| 15 | 2hPausada | Fall-Through     | := | Yes   |
| 16 | 2hPausada | Simultaneous-Use | := | 1     |
+----+-----------+------------------+----+-------+
```
## Ejemplo Perfil 1 Mes corrido
Este perfil lo utilizo al crear vouchers mensuales de un solo uso.
```
MariaDB [radius]> SELECT * FROM radgroupreply WHERE groupname = 'MENSUAL-1USO';
+----+--------------+-----------------------+----+---------------------------------------------+
| id | groupname    | attribute             | op | value                                       |
+----+--------------+-----------------------+----+---------------------------------------------+
| 37 | MENSUAL-1USO | Mikrotik-Rate-Limit   | := | 512K/2M 1M/3M 384K/1500K 16/12 8 256K/1000K |
| 26 | MENSUAL-1USO | Acct-Interim-Interval | := | 60                                          |
+----+--------------+-----------------------+----+---------------------------------------------+
2 rows in set (0.001 sec)

MariaDB [radius]> SELECT * FROM radgroupcheck WHERE groupname = 'MENSUAL-1USO';
+----+--------------+------------------+----+---------+
| id | groupname    | attribute        | op | value   |
+----+--------------+------------------+----+---------+
|  9 | MENSUAL-1USO | Access-Period    | := | 2592000 |
| 10 | MENSUAL-1USO | Fall-Through     | := | Yes     |
| 11 | MENSUAL-1USO | Simultaneous-Use | := | 1       |
+----+--------------+------------------+----+---------+
```
## Ejemplo Perfil Mensual recurrente
Este perfil me funciona para clientes a los cuales les vendo una mensualidad para un dispocitivo y la necesito renovar, ya que le creo un usuario y password personalizado.
- Se crea el perfil con los siguientes datos o atributos (radgroupcheck se refiere a atributos check) y (radgroupreply a los reply).

```
MariaDB [radius]> SELECT * FROM radgroupcheck WHERE groupname = '30dMensual';
+----+------------+------------------+----+-------+
| id | groupname  | attribute        | op | value |
+----+------------+------------------+----+-------+
| 17 | 30dMensual | Fall-Through     | := | Yes   |
| 18 | 30dMensual | Simultaneous-Use | := | 1     |
+----+------------+------------------+----+-------+
2 rows in set (0.001 sec)

MariaDB [radius]> SELECT * FROM radgroupreply WHERE groupname = '30dMensual';
+----+------------+-----------------------+----+----------------------------------------------------------------------+
| id | groupname  | attribute             | op | value                                                                |
+----+------------+-----------------------+----+----------------------------------------------------------------------+
| 32 | 30dMensual | Acct-Interim-Interval | := | 60                                                                   |
| 46 | 30dMensual | Mikrotik-Rate-Limit   | := | 512K/2M 1M/3M 384K/1500K 16/12 8 256K/1000K                          |
| 41 | 30dMensual | WISPr-Redirection-URL | =  | https://dailyverses.net/es/versiculo-de-la-biblia-al-azar/proteccion |
+----+------------+-----------------------+----+----------------------------------------------------------------------+
```
- Despues al crear unuevo usuario (SILVIA1), le colocamos nombre y passw
- Seleccionamos el perfil (30dMensual)
- en atributos agregamos un nuevo a**tributo** llamado `WISPr-Session-Terminate-Time` con **Op** `=` y **value** `2023-12-24T17:00:00` **tipo** `reply`
- Por lo que quedara asi.

```
MariaDB [radius]> SELECT * FROM radcheck WHERE username = 'SILVIA1';
+------+----------+--------------------+----+-------+
| id   | username | attribute          | op | value |
+------+----------+--------------------+----+-------+
| 1337 | SILVIA1  | Cleartext-Password | := | 45    |
+------+----------+--------------------+----+-------+

MariaDB [radius]> SELECT * FROM radreply WHERE username = 'SILVIA1';
+----+----------+------------------------------+----+---------------------+
| id | username | attribute                    | op | value               |
+----+----------+------------------------------+----+---------------------+
| 78 | SILVIA1  | WISPr-Session-Terminate-Time | =  | 2023-12-24T17:00:00 |
+----+----------+------------------------------+----+---------------------+
```

## lIMITAR AL USUARIO A UN SOLO NAS.
- Primero investigamos el nombre del NAS, para ello vamos al mikrotik y ejecutamos.
```
/system identity print
# El mio es RNvo5
```
- Ahora al usuario le agregamos un nuevo **atributo** llamado `NAS-Identifier` con **Op** `==` y **value** `RNvo5` <--El mio
- En nuestro servidor radius debemos buscar y cambiar a yes lo siguiente.
- Busca en tu servidor el archivo que contenga `copy_request_to_tunnel`

```
grep -rl "copy_request_to_tunnel" /etc
```
por ejemplo eso se encuentra en :
```
nano /etc/freeradius/3.0/mods-available/eap
```
- Edita el archivo y donde encuentres:
```
copy_request_to_tunnel = no
```
Cambialo a yes.
```
copy_request_to_tunnel = yes
```
Tambien cambia lo que diga 
```
use_tunneled_reply = no
```
igualmente a yes
```
use_tunneled_reply = yes
```
- Reinicia el servidor y ahora haz pruebas con tu usuario cambiando de NAS.

```
MariaDB [radius]> SELECT * FROM radcheck WHERE username = 'GARCIA1';
+------+----------+--------------------+----+-------+
| id   | username | attribute          | op | value |
+------+----------+--------------------+----+-------+
| 6357 | GARCIA1  | NAS-Identifier     | == | RNvo5 |
| 3194 | GARCIA1  | Cleartext-Password | := | 34    |
+------+----------+--------------------+----+-------+
2 rows in set (0.001 sec)
```
