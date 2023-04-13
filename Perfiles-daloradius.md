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
