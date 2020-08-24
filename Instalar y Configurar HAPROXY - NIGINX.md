# SistemasDistribuidos
## Instalar y Configurar HAPROXY – NGINX

loadbalancer	192.168.1.50 (haproxy)  
nginx1        192.168.1.51 (nginx-nodo01)  
nginx2        192.168.1.52 (nginx-nodo02)  

### HAPROXY
```
dnf install -y haproxy			//instalar haproxy
cd /etc/haproxy				//carpeta del haproxy
cp haproxy.cfg haproxy.cfg-orig		//crear copia de seguridad del archivo
vi haproxy.cfg				//editar el archivo
```
___
```
frontend http_balancer
	bind 192.168.1.50:80            //aquí va la ip del haproxy, cuando hay ipvirtual, se coloca *
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
### NGINX
```
dnf install -y nginx	//instalar nginx
systemctl start nginx
systemctl enable nginx
cd /usr/share/ngnix/html	//entrar entrar en la carpeta para crear un archivo html
```
___
```
echo "Nginx Node01 - Welcome to First Nginx Web Server" > index.html	//en los dos nodos
```
___
```
firewall-cmd --permanent --add-service=http	//habilitar el servicio http
firewall-cmd –reload	//cargar nuevamente el firewall
```
### VERIFICAR
```
curl 192.168.1.50	//en la ventana de comandos
```
http://192.168.1.50/haproxy?stats		//en el navegador  


