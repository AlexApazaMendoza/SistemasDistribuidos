# SistemasDistribuidos

## Instalar y Configurar KEEPALIVED - HAPROXY - NGINX
loadbalancerM	192.168.1.50 (haproxy-master)  
loadbalancerS	192.168.1.53 (haproxy-slave)  
nginx1		192.168.1.51 (nginx-nodo01)  
nginx2		192.168.1.52 (nginx-nodo02)  
ip virtual	192.168.1.49 (keepalived)  

### HAPROXY
Crear otra máquina virtual, y hacer la configuración del haproxy. (ojo: en bind colocar *:80)
### KEEPALIVED

dnf install -y keepalived	//instalar keepalived
cd /etc/ keepalived	//carpeta del keepalived
cp keepalived.conf keepalived.conf-orig	//crear copia de seguridad del archivo
vi keepalived.conf		//editar el archivo
-------------------------------------------------------------------	//HAPROXY MASTER
global_defs {
notification_email {
root@mydomain.com
}
notification_email_from svr1@mydomain.com
smtp_server localhost
smtp_connect_timeout 30
}
vrrp_instance VI_1 {
state MASTER
interface ens33	//Specify the network interface to which the virtual address is assigned
virtual_router_id 51	//The virtual router ID must be unique to each VRRP instance that you define
priority 100	//Set the value of priority higher on the master server than on a backup server
advert_int 1
authentication {
auth_type PASS
auth_pass 1111
}
virtual_ipaddress {
192.168.1.49/24	//ip virtual
}
}
-----------------------------------------------------------	//eliminar los demás
---------------------------------------------------------------	//HAPROXY SLAVE
global_defs {
notification_email {
root@mydomain.com
}
notification_email_from svr1@mydomain.com
smtp_server localhost
smtp_connect_timeout 30
}
vrrp_instance VI_1 {
state SLAVE
interface ens33	//Specify the network interface to which the virtual address is assigned
virtual_router_id 51	//The virtual router ID must be unique to each VRRP instance that you define
priority 101	//Set the value of priority higher on the master server than on a backup server
advert_int 1
authentication {
auth_type PASS
auth_pass 1111
}
virtual_ipaddress {
192.168.1.49/24	//ip virtual
}
}
-----------------------------------------------------------	//eliminar los demás
