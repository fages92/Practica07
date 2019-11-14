# Practica07
Balanceador Lemp
### En esta practica vamos a usar una maquina como balanceador de dos servidores nginx que acceden a un servidor mysql aparte

## Configuracion del Balanceador
### Vamos a usar un SCRIPT el cual nos configurara la maquina como balanceador
## SCRIPT
```
#!/bin/bash
#Actualizamos la maquina e instalamos ngix
sudo apt-get update
sudo apt-get install nginx -y

#instalacion de php-fpm, php-mysql y git
sudo apt-get install php-fpm php-mysql -y
apt-get install git -y

#configuracion de php-fpm
# con sed modificamos el archivo php.ini en /etc/php/7.2/fpm
cd /etc/php/7.2/fpm
sed -i 's/;cgi.fix_pathinfo=1/;cgi.fix_pathinfo=0/' php.ini
sudo systemctl restart php7.2-fpm

#configuracion de ngix
#Primero vamos a acceder a nuestro repositorio para obtener los archivos default y nginx.conf
cd /home/ubuntu
sudo rm -r /home/ubuntu/balanceador-lemp
sudo git clone https://github.com/fages92/balanceador-lemp.git
cd /home/ubuntu/balanceador-lemp
#Ahora vamos a sobreescribir nuestro archivo modificado nginx.conf en /etc/nginx
sudo mv nginx.conf /etc/nginx
#Ahora vamos a sobreescribir nuestro archivo modificado default en /etc/nginx/sites-available
sudo mv default /etc/nginx/sites-available
#Para que funcione debemos borrar el archivo default en /etc/nginx/sites-enabled/default y reiniciamos el servicio
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```
## Contenido del archivo default
### En este archivo hemos incluido las ips de las maquinas nginx e incluir que este servidor actua como balanceador
```
http {
    upstream backend {
        server 3.93.81.239;
        server 35.175.245.70;
    }
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        index index.php index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                autoindex on;
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
                proxy_pass http://backend;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                # With php7.2-fpm:
                fastcgi_pass unix:/run/php/php7.2-fpm.sock;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        location ~ /\.ht {
                deny all;
        }
}
```
## Contenido a modificar en el  Archivo nginx.conf
### Aqui indicamos tambien las ips de los servidores nginx y que nuestro balanceador va a actuar como tal, para ello debemos escribir lo siguiente dentro de html
```
upstream backend {
        server 3.93.81.239;
        server 35.175.245.70;
    }
server {
        location / {
            proxy_pass http://backend;
        }
    }
```
## El repositorio de estos archivos es el siguiente: https://github.com/fages92/balanceador-lemp.git
