# Install Apache2 
```
apt update 
apt install apache2
```
# Apache status 
```
service apache2 status 
```

# Install PHP
```
apt install php libapache2-mod-php
apt install php7.2-mysql php7.2-curl php7.2-json php7.2-cgi php7.2-xsl
service apache2 restart
```
# Upgrade PHP
```
  add-apt-repository ppa:ondrej/php
  apt-get update
  apt install php7.3 
  apt install php7.3-mysql php7.3-curl php7.3-json php7.3-cgi php7.3-xsl
  service apache2 restart
  a2dismod php7.2
  a2enmod php7.3
  service apache2 restart
  php -v
```
