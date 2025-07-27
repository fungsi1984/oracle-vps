### some notes

```
from official doc
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo netfilter-persistent save

ip tables in oracle doc not works, this temporary solution
sudo iptables -I INPUT 1 -p tcp --dport 80 -j ACCEPT
sudo netfilter-persistent save


explaination, we need to put line 6 before reject rules in line 5
sudo iptables -L INPUT -n --line-numbers
5    REJECT     0    --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited
6    ACCEPT     6    --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:80

# Delete the current rule (line 6)
sudo iptables -D INPUT 6

# Add the rule at position 5 (before the REJECT rule)
sudo iptables -I INPUT 5 -m state --state NEW -p tcp --dport 80 -j ACCEPT

# Save the rules
sudo netfilter-persistent save

curl -i 158.180.5.145

```

### check ssl/tls
```
sudo certbot --nginx certonly -d diacritics.xyz -d www.diacritics.xyz
ubuntu@instance-20250727-1646:~$ sudo certbot renew --dry-run
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/diacritics.xyz.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Account registered.
Simulating renewal of an existing certificate for diacritics.xyz and www.diacritics.xyz

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded: 
  /etc/letsencrypt/live/diacritics.xyz/fullchain.pem (success)



```
### nginx conf, /etc/nginx/sites-available/diacritics.xyz
```
server {
    listen 80;
    listen [::]:80;
    server_name diacritics.xyz www.diacritics.xyz;
    
    return 301 https://diacritics.xyz$request_uri;
}

# HTTPS server block - main site
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name diacritics.xyz;

    # Real Let's Encrypt certificates
    ssl_certificate /etc/letsencrypt/live/diacritics.xyz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/diacritics.xyz/privkey.pem;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_index index.php;
        include fastcgi.conf;
    }
}

# HTTPS server block - www redirect
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name www.diacritics.xyz;

    # Real Let's Encrypt certificates
    ssl_certificate /etc/letsencrypt/live/diacritics.xyz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/diacritics.xyz/privkey.pem;

    return 301 https://diacritics.xyz$request_uri;
}

```



### configure for cloudflare
```
Type: A
Name: @
Content: 158.180.5.145
Proxy status: DNS only (gray cloud)
TTL: Auto


Type: CNAME
Name: www
Content: diacritics.xyz
Proxy status: DNS only (gray cloud)
TTL: Auto
```
### configure security oracle cloud
```
add ingress for port 443
Add Ingress Rule:

    Source CIDR: 0.0.0.0/0 (or be more specific for security)
    IP Protocol: TCP
    Source Port Range: All
    Destination Port Range: 443
    Description: Allow HTTPS

```  
