# This Document only for UBUNTU 18.04

# Uninstall 

```apt remove prosody
apt remove --auto-remove prosody
apt purge prosody  
apt purge apache2
apt remove nginx nginx-common
apt purge nginx nginx-common
apt purge jigasi jitsi-meet jitsi-meet-web-config jitsi-meet-prosody jitsi-meet-turnserver jitsi-meet-web jicofo jitsi-videobridge2
apt autoremove
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
apt install certbot python3-certbot-apache
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
apt-get update -y &&
apt-get install gcc -y &&
apt-get install unzip -y &&
apt-get install lua5.2 -y &&
apt-get install liblua5.2 -y &&
apt-get install luarocks -y &&
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
wget https://prosody.im/files/prosody-debian-packages.key -O- | sudo apt-key add - &&
echo deb http://packages.prosody.im/debian $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list &&
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
apt install jitsi-meet-tokens 
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
rm /etc/apache2/sites-enabled/000-default.conf
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
        anonymousdomain: 'guest.[YOUR DOMAIN]',
        ...
    },
    ...
    enableUserRolesBasedOnToken: true,
    ...
}
```

### Edit jicofo sip-communicator in `/etc/jitsi/jicofo/sip-communicator.properties`
```
org.jitsi.jicofo.auth.URL=XMPP:[YOUR DOMAIN]
org.jitsi.jicofo.auth.DISABLE_AUTOLOGIN=true
```

### Edit jicofo config in `/etc/jitsi/jicofo/config`
SET the follow configs
```
JICOFO_HOST=[YOUR DOMAIN]
```

### And edit videobridge config in `/etc/jitsi/videobridge/config`

Replace 
```
JVB_HOST=
```
TO
```
JVB_HOST=[YOUR DOMAIN]
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

npm install or npm i

```
To create build on localhost
```
make dev 
```
To create build on live
```
make 
```

## Installing MySQL Client And Lua-DBI-MySQL

For Installing MySQL CLient 

```
apt install mysql-client-core-5.7  

```

Login into MySQL

```
 mysql -u [Database_Username] -h [hostname] -p
 
```

For Installing Lua-DBI-MySQL

```

apt install lua-dbi-common lua-dbi-mysql

```

# Configuring Prosody with mysql

Edit `/etc/prosody/prosody.cfg.lua`

Uncomment, ` "mam"; -- Store messages in an archive and allow users to access it ` under `modules_enabled = {`

```

modules_enabled = {
...
 -- Nice to have
 ...
 "mam"; -- Store messages in an archive and allow users to access it
 ...
 
```

Changing Storage location

```

storage = "internal"

```

To

```

storage = "sql" 

```

Uncomment ` sql = { driver = "MySQL", database = "[DatabaseName]", username = "[DatabaseUsername]", password = "[DatabasePassword]", host = "[Hostname]" } `


Add users as per config 

```
prosodyctl adduser jvb@auth.[YOUR DOMAIN]
prosodyctl adduser focus@auth.[YOUR DOMAIN]
prosodyctl adduser jigasi@auth.[YOUR DOMAIN] (if enabled)
```

Open `/etc/prosody/conf.avail/[YOUR DOMAIN].cfg.lua`
and add mod for conference component 

```
muc_mam
```
Finally exit the editor and restart services.

```

systemctl restart apache2 prosody jicofo jitsi-videobridge2

````

# Jibri Instalation & Configuration

For jibri instalation we would require a seprate server with minimum 2 CPUs & 4 GB RAM, and good storage space for recordings.


## Jibri Instalation 

Check if the server's Operating System is using a Generic Kernel or not

```
uname -r
```

If its not using a generic kernel then , you have to change the Kernel to Generic.

# Changing Kernel

Edit `/etc/default/grub `

Change 

```
GRUB_DEFAULT="..."
```
to 

```
GRUB_DEFAULT="1>4"
```

Exit the edit mode and update GRUB.

```
 update-grub
 reboot
```

Installing Generic Kernel

```
apt update
apt install linux-image-extra-virtual ffmpeg curl unzip software-properties-common
```

Removing Old Kernel

For getting Kernel info

```
grep -A200 submenu /boot/grub/grub.cfg | grep menuentry
```

For Removing Old Kernel 

```
apt remove [Old-Kernel-Name] [Old-Kernel-Recovery-Name] 
systemctl reboot
```

# Installing Jibri 

Installing Sound Modules 

```
echo "snd_aloop" >> /etc/modules
modprobe snd_aloop 
```
For checking if module is installed or not 

```
lsmod | grep snd_aloop
```

Installing Additional Packages

```
curl -sS -o - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add
echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chromewget -N 
apt update
apt install google-chrome-stable
CHROME_DRIVER_VERSION=`curl -sS chromedriver.storage.googleapis.com/LATEST_RELEASE`
wget -N http://chromedriver.storage.googleapis.com/$CHROME_DRIVER_VERSION/chromedriver_linux64.zip -P ~/
unzip ~/chromedriver_linux64.zip -d ~/
rm ~/chromedriver_linux64.zip
mv -f ~/chromedriver /usr/local/bin/chromedriver
chmod 0755 /usr/local/bin/chromedriver
mkdir -p /etc/opt/chrome/policies/managed
echo '{ "CommandLineFlagSecurityWarningsEnabled": false }' >>/etc/opt/chrome/policies/managed/managed_policies.json
add-apt-repository ppa:mc3man/trusty-media
wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add -
sh -c "echo 'deb https://download.jitsi.org stable/' > /etc/apt/sources.list.d/jitsi-stable.list"
```

Installing Jibri 

```
apt update
apt install jibri
```

# Configuring Jibri

```
usermod -aG adm,audio,video,plugdev jibri
```

Giving Path for Recordings

```
mkdir /recordings
chown jibri:jibri /recordings
```

Edit ` /etc/jitsi/jibri/config.json `

```
"recording_directory": "[path for recordings directory]",
"xmpp_server_hosts": [
"[YOUR DOMAIN]"
],
"xmpp_domain": "[YOUR DOMAIN]",
"control_login": {
"domain": "auth.[YOUR DOMAIN]",
"username": "[jibri_username]",
"password": "[jibri_password]"
},
"control_muc": {
"domain": "internal.auth.[YOUR DOMAIN]",
"room_name": "JibriBrewery",
"nickname": "jibri"
},
"call_login": {
"domain": "recorder.[YOUR DOMAIN]",
"username": "[Recorder_Username]",
"password": "[Recorder_password]"
}, 

```

Jibri only works on Java 8

Java 8 Configuration

```
wget -O - https://adoptopenjdk.jfrog.io/adoptopenjdk/api/gpg/key/public | sudo apt-key add -
add-apt-repository https://adoptopenjdk.jfrog.io/adoptopenjdk/deb/
apt update
apt install adoptopenjdk-8-hotspot
```
Edit `/opt/jitsi/jibri/launch.sh` , replace `java` with `/usr/lib/jvm/adoptopenjdk-8-hotspot-amd64/bin/java`

Finally,

```
systemctl enable --now jibri
systemctl restart jibri
```

## Jitsi Configuration ( On Jitsi Server )

Edit `/etc/prosody/conf.avail/[YOUR DOMAIN].cfg.lua `

Add ` muc_mam `

```
Component "conference.[YOUR DOMAIN]" "muc"
modules_enabled = { .
                    .
                    "muc_mam"
                    } 
```

Add ` muc_room_cache_size = 1000 ` 

```
Component "internal.auth.[YOUR DOMAIN]" "muc"
modules_enabled = {
"ping";
}
storage = "internal"
muc_room_cache_size = 1000 
```

Add a new `VirtualHost` for Recorder

```
VirtualHost "recorder.[YOUR DOMAIN]"
modules_enabled = {
"ping";
}
authentication = "internal_plain"
```
Save and Exit

Create two new accounts for Jibri to use ( For Recorder and Jibri )

```
prosodyctl register [Jibri_Username] auth.[YOUR DOMAIN][Jibri_Password]
prosodyctl register [Recorder_Username] recorder.[YOUR DOMAIN][Recorder_Password]
```

Edit `/etc/jitsi/jicofo/sip-communicator.properties`, and Add

```
org.jitsi.jicofo.jibri.BREWERY=JibriBrewery@internal.auth.[YOUR DOMAIN]
org.jitsi.jicofo.jibri.PENDING_TIMEOUT=90
```

Edit ` /etc/jitsi/meet/[YOUR DOMAIN]-config.js`

Change and Uncomment ` --fileRecordingsEnabled: false `
                  To ` fileRecordingsEnabled: true, `
                  
Change and Uncomment ` --liveStreamingEnabled: false `
                  To ` liveStreamingEnabled: true, `
                  
Add `hiddenDomain: 'recorder.[YOUR DOMAIN]',`

Save and Exit

Finally Restart All Services

```
systemctl restart prosody jicofo jitsi-videobridge2 jigasi

```

# Load Balancing

## Videobridge

### Prerequisite:

Server OS - ubuntu 18.04 | Ports - 443/tcp , 4443/tcp , 10000:20000/udp

### Installing Videobridge on Videobridge Server

Installing Videobridge

```
echo 'deb https://download.jitsi.org stable/' >> /etc/apt/sources.list.d/jitsi-stable.list
wget -qO -  https://download.jitsi.org/jitsi-key.gpg.key | apt-key add -
apt update
apt install jitsi-videobridge2
```

### Configuring Videobridge

Edit `/etc/jitsi/videobridge/config`

Change `JVB_HOST=`
To `JVB_HOST=[hostname]`

Change `JVB_SECRET=xyz123`
To `JVB_SECRET=[JVB_Jitsi_Server_Secret]`

Find `[JVB_Jitsi_Server_Secret]` in `/etc/jitsi/videobridge/config` on jitsi server.

Add `AUTHBIND=yes` at the end of the file.

Save & Exit

Edit `/etc/jitsi/videobridge/sip-communicator.properties`

Change `org.jitsi.videobridge.xmpp.user.shard.HOSTNAME=localhost`
To `org.jitsi.videobridge.xmpp.user.shard.HOSTNAME=[Hostname]`

Change `org.jitsi.videobridge.xmpp.user.shard.PASSWORD=xyz123`
To `org.jitsi.videobridge.xmpp.user.shard.PASSWORD=[JVB_Jitsi_Server_Secret]`

Find `[JVB_Jitsi_Server_Secret]` in `/etc/jitsi/videobridge/config` on jitsi server.

Add `org.jitsi.videobridge.xmpp.user.shard.DISABLE_CERTIFICATE_VERIFICATION=true`

Save & Exit

#### Restarting Videobridge

```
systemctl restart jitsi-videobridge2
```
#### Videobridge Status

```
systemctl status jitsi-videobridge2
```

#### Videobridge Logs

```
tail -f /var/log/jitsi/jvb.log
```
