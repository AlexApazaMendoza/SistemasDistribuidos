# SistemasDistribuidos
## Instalar y configurar clúster Mariadb Galera

nodo1		192.168.1.91 (mariadb-galera-nodo01)  
nodo2		192.168.1.92 (mariadb-galera -nodo02)  
nodo3		192.168.1.93 (mariadb-galera -nodo03)  

### MARIADB - GALERA
En los 3 servidores:
```
dnf install -y mariadb-server-galera	//instalar servidores
firewall-cmd --add-port={3306/tcp,4567/tcp,4568/tcp,4444/tcp} --permanent	//habilitar los puertos
firewall-cmd –reload	//cargar nuevamente el firewall
```
Configurar en el servidor 1:  
```
vi /etc/my.cnf.d/galera.cnf	//carpeta donde está el archivo de configuración
```
descomentar las líneas y agregar
```
wsrep_cluster_name="Galera_Cluster"
wsrep_cluster_address="gcomm://192.168.1.91,192.168.1.92,192.168.1.93"	//ips de los nodos
wsrep_node_address="192.168.1.91"	//ip del nodo
```
___
```
galera_new_cluster	//activar el clúster 
systemctl enable mariadb	//inicie al encender la máquina virtual
mysql_secure_installation	//iniciar con una configuración segura
```
OBS: al no tener contraseña, se presiona enter (se deja vacío), se establece una contraseña, y luego a todas las demás preguntas, colocar Y (yes).  
Configurar en el servidor 2:  
```
vi /etc/my.cnf.d/galera.cnf	//carpeta donde está el archivo de configuración
```
descomentar las líneas y agregar  
```
wsrep_cluster_name="Galera_Cluster"
wsrep_cluster_address="gcomm://192.168.1.91,192.168.1.92,192.168.1.93"
wsrep_node_address="192.168.1.92"	//ip del nodo
```
___
```
systemctl enable --now mariadb	//el parámetro now es para que inicialice
```
Configurar en el servidor 3:   
```
vi /etc/my.cnf.d/galera.cnf	//carpeta donde está el archivo de configuración
```
descomentar las líneas y agregar   
```
wsrep_cluster_name="Galera_Cluster"
wsrep_cluster_address="gcomm://192.168.1.91,192.168.1.92,192.168.1.93"
wsrep_node_address="192.168.1.93"	//ip del nodo
```
___
```
systemctl enable --now mariadb	//el parámetro now es para que inicialice
```
### ENTRAR A MARIADB
Entrar a mariadb:   
```
mysql -u root -p
```
### ALGUNOS COMANDOS:```
show databases;	//mostrar las bases de datos en el clúster
create database test1;	//crear una base de datos
```


