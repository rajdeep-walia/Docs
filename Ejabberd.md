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
apt install build-essential

Libexpat 1.95 or higher
Libyaml 0.1.4 or higher
Erlang/OTP 19.3 or higher. We recommend using Erlang OTP 21.2.
OpenSSL 1.0.0 or higher, for STARTTLS, SASL and SSL encryption.
Zlib 1.2.3 or higher, for Stream Compression support (XEP-0138). Optional.
PAM library. Optional. For Pluggable Authentication Modules (PAM). See PAM Authentication section.
ImageMagick’s Convert program and Ghostscript fonts. Optional. For CAPTCHA challenges. See section CAPTCHA.
```
