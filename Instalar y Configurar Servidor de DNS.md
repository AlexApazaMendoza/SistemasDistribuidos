# SistemasDistribuidos

## Instalar y Configurar Servidor de DNS
serverDNS	serverdns	192.168.1.200  
Mail		mail		192.168.1.201  
Pc01		pc01		192.168.1.211  
www		www		192.168.1.202  

### SERVERDNS
Actualizar Centos7:  
```
yum update && upgrade
```
Instalar dns bind:  
```
yum -y install bind*
```
Desactivar el firewall:  
```
systemctl stop firewalld
systemctl disable firewalld
```
Desactivar Selinux:  
```
vi /etc/selinux/config
```
___
```
SELINUX=disabled		//cambiar a disabled
```
___
Configurar archivo:  
```
vi /etc/named.conf
```
___
```
options {
        listen-on port 53 { 127.0.0.1; 192.168.1.200 };	//ip del dns
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { localhost; 192.168.1.0/24; }; //mascara de red
        ...
        //Bajamos hasta el final
        //Comentamos la línea 57, queda así como abajo
        ...
zone "." IN {
        type hint;
        file "named.ca";
};

zone "distribuidos.lr" IN {   //colocamos el dominio
        type master;
        file "directa.distribuidos.lr.zone";
        allow-update { none; };
};

zone "1.168.192.in-addr.arpa" IN {   //colocamos el dominio
        type master;
        file "inversa.distribuidos.lr.zone";
        allow-update { none; };
}; 
```
Configuramos la carpeta directa  
```
vi /var/named/directa.distribuidos.lr.zone
```
___
```
$TTL 1D
@   IN SOA  serverdns.distribuidos.lr.      root.distribuidos.lr. (
                                        1   ; serial
                                        1H  ; refresh
                                        1H  ; retry
                                        1W  ; expire
                                        3H )    ; minimum

localhost               1H  A   127.0.0.1
distribuidos.lr.        1H  NS  serverdns.distribuidos.lr.
distribuidos.lr.        1H  MX  10 mail.distribuidos.lr.
serverdns               1H  A   192.168.1.200
pc01                    1H  A   192.168.1.211
mail                    1H  A   192.168.1.201
www                     1H  A   192.168.1.202
```
Configuramos la carpeta inversa:  
```
vi /var/named/inversa.distribuidos.lr.zone
```
___
```
$TTL 1D
@   IN SOA  serverdns.distribuidos.lr.      root.distribuidos.lr. (
                                        1   ; serial
                                        1H  ; refresh
                                        1H  ; retry
                                        1W  ; expire
                                        3H )    ; minimum

                        IN  NS  serverdns.distribuidos.lr.
254                     IN  PTR serverdns.distribuidos.lr.
250                     IN  PTR mail.distribuidos.lr.
100                     IN  PTR pc01.distribuidos.lr.
254                     IN  PTR www.distribuidos.lr.
```
Configurar la red:  
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```
___
```
BOOTPROTO="static"
...
...
...
DNS1="192.168.1.200"
```
Reiniciar la red:  
```
systemctl restart network
```
Habilitar el dns:  
```
systemctl enable named
```
Iniciar el dns:  
```
systemctl start named
```
### VERIFICAR
```
nslookup
dig serverdns.distribuidos.lr
```
