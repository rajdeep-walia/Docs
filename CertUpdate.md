# Lests encrypt cert update 

```
cd /etc/letsencrypt/live/[YOUR DOMAIN]
openssl x509 -enddate -noout -in cert.pem
```

# Update cert
```
certbot renew
```
