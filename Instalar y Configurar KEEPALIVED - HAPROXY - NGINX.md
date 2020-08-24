# SistemasDistribuidos

## Instalar y Configurar KEEPALIVED - HAPROXY - NGINX
loadbalancerM	192.168.1.50 (haproxy-master)  
loadbalancerS	192.168.1.53 (haproxy-slave)  
nginx1		192.168.1.51 (nginx-nodo01)  
nginx2		192.168.1.52 (nginx-nodo02)  
ip virtual	192.168.1.49 (keepalived)  

### HAPROXY
```
dnf install -y haproxy			//instalar haproxy
cd /etc/haproxy				//carpeta del haproxy
cp haproxy.cfg haproxy.cfg-orig		//crear copia de seguridad del archivo
vi haproxy.cfg				//editar el archivo
```
#### HAPROXY MASTER
```
frontend http_balancer
	bind *:80            //aquí va la ip del haproxy, cuando hay ipvirtual, se coloca *
	option http-server-close
	option forwardfor
	stats uri /haproxy?stats
#    	acl url_static       path_beg       -i /static /images /javascript /stylesheets
#    	acl url_static       path_end       -i .jpg .gif .png .css .js

#    	use_backend static          if url_static
	default_backend nginx_webservers
backend nginx_webservers
	mode http
	balance roundrobin               //tipo de algoritmo
	option httpchk HEAD / HTTP/1.1\r\nHost:\ localhost
	server nginx1 192.168.1.51:80	check
	server nginx2 192.168.1.52:80	check
```
#### HAPROXY SLAVE
```
frontend http_balancer
	bind *:80            //aquí va la ip del haproxy, cuando hay ipvirtual, se coloca *
	option http-server-close
	option forwardfor
	stats uri /haproxy?stats
#    	acl url_static       path_beg       -i /static /images /javascript /stylesheets
#    	acl url_static       path_end       -i .jpg .gif .png .css .js

#    	use_backend static          if url_static
	default_backend nginx_webservers
backend nginx_webservers
	mode http
	balance roundrobin               //tipo de algoritmo
	option httpchk HEAD / HTTP/1.1\r\nHost:\ localhost
	server nginx1 192.168.1.51:80	check
	server nginx2 192.168.1.52:80	check
```
Configure rsyslog para que almacene todas las estadísticas de HAProxy, entrar al archivo “/etc/rsyslog.conf” y descomentar las líneas 19 y 20:  
```
vi /etc/rsyslog.conf	//entrar al archivo
```
___
```
module(load="imudp")
input(type="imudp" port="514")
```
___
```
systemctl restart rsyslog
systemctl enable rsyslog
```
Now finally start haproxy but before starting haproxy service, set the following selinux rule:  
```
setsebool -P haproxy_connect_any 1
systemctl start haproxy
systemctl enable haproxy
firewall-cmd --permanent --add-port=80/tcp		//habilitar el puerto 80
firewall-cmd –reload					//cargar nuevamente el firewall
```
### KEEPALIVED
```
dnf install -y keepalived                 //instalar keepalived
cd /etc/ keepalived                       //carpeta del keepalived
cp keepalived.conf keepalived.conf-orig   //crear copia de seguridad del archivo
vi keepalived.conf                        //editar el archivo
```
#### HAPROXY MASTER
```
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
```
#### HAPROXY SLAVE
```
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
```
Habilitar el reenvío de IP:
```
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p	// net.ipv4.ip_forward = 1
systemctl start keepalived
systemctl enable keepalived
```
Si cambia la configuración de Keepalived, vuelva a cargar el servicio keepalived:  
```
systemctl reload keepalived
```
### VERIFICAR
http://192.168.1.49/haproxy?stats		//en el navegador  
