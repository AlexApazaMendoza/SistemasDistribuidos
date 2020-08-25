# SistemasDistribuidos

## Conectar haproxy – clúster mariadb galera

server1	192.168.1.81 (haproxy)  
server2	192.168.1.82 (haproxy)  
ip virtual	192.168.1.90 (keepalived)  
nodo1		192.168.1.91 (mariadb-galera-nodo01)  
nodo2		192.168.1.92 (mariadb-galera -nodo02)  
nodo3		192.168.1.93 (mariadb-galera -nodo03)  

### HAPROXY
```
dnf install -y haproxy  //ya estaba instalado
cd /etc/haproxy         //ir a la carpeta haproxy
vi haproxy.cfg          //abrir el archivo de configuración
```
___
```
listen stats
  bind    *:9000
  mode    http
  stats   enable
  stats   realm Haproxy\ Statistics
  stats   uri /haproxy?stats
  stats   auth admin:admin
listen galera
  bind    *:3306
  mode    tcp
  balance roundrobin
  option mysql-check user cluster_galera  //usuario con todos los privilegios creado en el cluster,en el cual se va a conectar el haproxy
  server nodo1 192.168.1.91:3306 check weight 1
  server nodo1 192.168.1.92:3306 check weight 1
  server nodo1 192.168.1.93:3306 check weight 1
```
___
```
systemctl restart haproxy
```
### Nodo1
El clúster ya está configurado en alta disponibilidad.  
Ahora creo un usuario en el cual el haproxy se va a conectar (en nuestro caso es cluster_galera, éste nombre va en el archivo de configuración de haproxy), por lo que le doy todos los privilegios.  
```
mysql -u root -p
create user ‘cluster_galera’@’%’;	//no le pongo contraseña, % todas las ip
grant all privileges on *.* to ‘cluster_galera’@’%’;	//cluster_galera es el nombre
flush privileges; //aplicar los cambios
select host, user from mysql.user;	//para visualizar los usuarios
```
Como los nodos están sincronizados, solo se hace en uno de los nodos.  
### Verificar
http://192.168.1.90:9000/haproxy?stats	//le puse la ip virtual  
