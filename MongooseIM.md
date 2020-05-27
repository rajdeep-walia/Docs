# Install Apache2 
```
apt update 
apt install apache2
```
# Install certbot 
```
apt install software-properties-common
add-apt-repository universe
add-apt-repository ppa:certbot/certbot
apt update
apt-get install certbot python3-certbot-apache
```
# Generate certs 
```
certbot --apache -d [YOUR DOMAIN]
```
Copy paths of key and cert:
```
/etc/letsencrypt/live/[YOUR DOMAIN]/privkey.pem
/etc/letsencrypt/live/[YOUR DOMAIN]/fullchain.pem
```
We need to set them on ejabberd yml file. 

# Pre Requirements
To compile ejabberd on a ‘Unix-like’ operating system, you need:
```
apt install make
apt install gcc
apt install g++
apt install erlang
apt install libssl-dav
apt install unixodbc-dav
apt install zlibig-dav
```
# Clone and build Ejabberd code
```
git clone https://github.com/esl/MongooseIM.git
cd MongooseIM
make
make rel
```
# Check if its working 
```
_build/prod/rel/mongooseim/mongooseimctl start
_build/prod/rel/mongooseim/mongooseimctl status
```
