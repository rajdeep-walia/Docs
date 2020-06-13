# Uninstall 

```apt remove prosody
apt remove --auto-remove prosody
apt purge prosody  
apt purge apache2
apt-get remove nginx nginx-common
apt-get purge nginx nginx-common
apt purge jigasi jitsi-meet jitsi-meet-web-config jitsi-meet-prosody jitsi-meet-turnserver jitsi-meet-web jicofo jitsi-videobridge2
apt-get autoremove
```
# Set Host name 
```
hostnamectl set-hostname [YOUR DOMAIN]
vi /etc/hosts
127.0.0.1  [YOUR DOMAIN]
ls -l /etc/cloud/cloud.cfg
vi /etc/cloud/cloud.cfg
preserve_hostname: true
hostnamectl
```
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
Jitsi meet will ask you to fill them while installing 
# Install lua and dependencies

Enter in sudo user

```bash
sudo su
```

Run the follow script to install lua and her dependencies, and prosody with fixes.
After finish, VM will be restarted
```bash
cd &&
apt update -y &&
apt install git -y &&
apt install gcc -y &&
apt install unzip -y &&
apt install lua5.2 -y &&
apt install liblua5.2 -y &&
apt install luarocks -y &&
luarocks install basexx &&
apt-get install libssl1.0-dev -y &&
luarocks install luacrypto &&
mkdir src &&
cd src &&
luarocks download lua-cjson &&
luarocks unpack lua-cjson-2.1.0.6-1.src.rock &&
cd lua-cjson-2.1.0.6-1/lua-cjson &&
sed -i 's/lua_objlen/lua_rawlen/g' lua_cjson.c &&
sed -i 's|$(PREFIX)/include|/usr/include/lua5.2|g' Makefile &&
luarocks make &&
luarocks install luajwtjitsi &&
cd &&
wget https://prosody.im/files/prosody-debian-packages.key -O- |  apt-key add - &&
echo deb http://packages.prosody.im/debian $(lsb_release -sc) main |  tee -a /etc/apt/sources.list &&
apt-get update -y &&
apt-get upgrade -y &&
apt-get install prosody -y &&
chown root:prosody /etc/prosody/certs/localhost.key &&
chmod 644 /etc/prosody/certs/localhost.key &&
sleep 2 &&
shutdown -r now
```

# Install jitsi-meet

Enter in sudo user
```bash
sudo su
```

After reboot, run the following commands:
```
wget -qO - https://download.jitsi.org/jitsi-key.gpg.key |  apt-key add - &&
sh -c "echo 'deb https://download.jitsi.org stable/' > /etc/apt/sources.list.d/jitsi-stable.list" &&
apt update
apt install jitsi-meet
apt-get install jitsi-meet-tokens 
```
# Open necessary ports

Enable ufw 
```bash
ufw enable
```

Open ports
```bash
ufw allow in 22/tcp &&
ufw allow in openssh &&
ufw allow in 80/tcp &&
ufw allow in 443/tcp &&
ufw allow in 4443/tcp &&
ufw allow in 5222/tcp &&
ufw allow in 5347/tcp &&
ufw allow in 10000:20000/udp
```
```
rm /etc/apache2/sites-enabled/000-default-le-ssl.conf 
service apache2 restart
```
# Config prosody

Open `/etc/prosody/prosody.cfg.lua` and

Add above lines after admins object
```
admins = {}

component_ports = { 5347 }
component_interface = "0.0.0.0"
```
 
 change 
```
c2s_require_encryption=true
```
to
```
c2s_require_encryption=false
```

and check if on end of file has
```
Include "conf.d/*.cfg.lua"
```

# Prosody manual plugin configuration

### Setup issuers and audiences 

Open `/etc/prosody/conf.avail/[YOUR DOMAIN].cfg.lua`

and add above lines with your issuers and audiences

```
asap_accepted_issuers = { "jitsi", "smash" }
asap_accepted_audiences = { "jitsi", "smash" }
```

### Under you domain config change authentication to "token" and provide application ID, secret and optionally token lifetime:

```
VirtualHost "jitmeet.example.com"
    authentication = "token";
    app_id = "example_app_id";             -- application identifier
    app_secret = "example_app_secret";     -- application secret known only to your token
```

### To access the data in lib-jitsi-meet you have to enable the prosody module mod_presence_identity in your config.

```
VirtualHost "jitmeet.example.com"
    modules_enabled = { "presence_identity" }
```

### Enable room name token verification plugin in your MUC component config section:

```
Component "conference.jitmeet.example.com" "muc"
    modules_enabled = { "token_verification" }
```

### Setup guest domain
```
VirtualHost "guest.[YOUR Domain]"
    authentication = "token";
    app_id = "example_app_id";
    app_secret = "example_app_secret";
    c2s_require_encryption = true;
    allow_empty_token = true;
```

### Enable guest domain in config.js
Open your meet config in `/etc/jitsi/meet/<host>-config.js` and enable
```js
var config = {
    hosts: {
        ... 
        // When using authentication, domain for guest users.
        anonymousdomain: 'guest.jitmeet.example.com',
        ...
    },
    ...
    enableUserRolesBasedOnToken: true,
    ...
}
```

### Edit jicofo sip-communicator in `/etc/jitsi/jicofo/sip-communicator.properties`
```
org.jitsi.jicofo.auth.URL=XMPP:jitmeet.example.com
org.jitsi.jicofo.auth.DISABLE_AUTOLOGIN=true
```

### Edit jicofo config in `/etc/jitsi/jicofo/config`
SET the follow configs
```
JICOFO_HOST=jitmeet.example.com
```

### And edit videobridge config in `/etc/jitsi/videobridge/config`

Replace 
```
JVB_HOST=
```
TO
```
JVB_HOST=jitmeet.example.com
```

And add after `JAVA_SYS_PROPS`
```
JAVA_SYS_PROPS=...
AUTHBIND=yes
```

Then, restart all services
```bash
systemctl restart apache2 prosody jicofo jitsi-videobridge2
```

# Setup jigasi for sip call
Be ready with sip username and password 
```
apt install jigasi
```
vi /etc/jitsi/jigasi/sip-communicator.properties and add
```
net.java.sip.communicator.impl.protocol.sip.SKIP_REINVITE_ON_FOCUS_CHANGE_PROP=true
net.java.sip.communicator.impl.protocol.sip.acc1403273890647.JITSI_MEET_ROOM_HEADER_NAME=X-Jitsi-Conference-Room
```
Also Add
```
org.jitsi.jigasi.xmpp.acc.USER_ID=jigasi@auth.[YOUR DOMAIN]
org.jitsi.jigasi.xmpp.acc.PASS=[Password]
org.jitsi.jigasi.xmpp.acc.ANONYMOUS_AUTH=false
```
Register user :
```
prosodyctl adduser jigasi@auth.[Your Domain]
```
vi /etc/jitsi/meet/[YOUR DOMAIN]-config.js and enable
```
dialInNumbersUrl: '[Your numbers to be enabled]',
dialInConfCodeUrl: 'https://jitsi-api.jitsi.net/conferenceMapper', 
```
# Helpers

Restart all services
```bash
systemctl restart prosody jicofo jitsi-videobridge2 jigasi
```

Prosody logs
```bash
tail -f -n 350 /var/log/prosody/prosody.log
```

Jicofo logs
```bash
tail -f -n 350 /var/log/jitsi/jicofo.log
```

Jigasi logs
```bash
tail -f -n 350 /var/log/jitsi/jigasi.log
```

JVB logs
```bash
tail -f -n 350 /var/log/jitsi/jvb.log
```



# Manual react-jitsi-meet

## Install NodeJS

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo bash - &&
apt-get install nodejs -y

```

## Install Jitsi-meet using React

```
git clone https://github.com/jitsi/jitsi-meet

git checkout  532dadb245c749f93e3c9ded8e66f17196ced15b

npm install or npm i

git checkout master 
```
To create build on localhost
```
make dev 
```
To create build on live
```
make 
```

