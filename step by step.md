### configure security oracle cloud
```
add ingress for port 80, for http
Add Ingress Rule:

    Source CIDR: 0.0.0.0/0 (or be more specific for security)
    IP Protocol: TCP
    Source Port Range: All
    Destination Port Range: 80
    Description: Allow HTTP

add ingress for port 443, for https
Add Ingress Rule:

    Source CIDR: 0.0.0.0/0 (or be more specific for security)
    IP Protocol: TCP
    Source Port Range: All
    Destination Port Range: 443
    Description: Allow HTTPS

```  

sudo apt update

sudo apt install nginx -y

sudo iptables -L INPUT -n --line-numbers

Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
2    ACCEPT     1    --  0.0.0.0/0            0.0.0.0/0           
3    ACCEPT     0    --  0.0.0.0/0            0.0.0.0/0           
4    ACCEPT     6    --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
5    REJECT     0    --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

sudo iptables -I INPUT 5 -m state --state NEW -p tcp --dport 80 -j ACCEPT

sudo netfilter-persistent save

sudo iptables -I INPUT 5 -m state --state NEW -p tcp --dport 443 -j ACCEPT

sudo netfilter-persistent save

nslookup word.diacritics.xyz
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   word.diacritics.xyz
Address: 152.67.108.218

curl -I http://152.67.108.218
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)

sudo nano /etc/nginx/sites-available/word.diacritics.xyz

server {
    listen 80;
    server_name word.diacritics.xyz;

    root /var/www/word.diacritics.xyz;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

sudo mkdir -p /var/www/word.diacritics.xyz
sudo chown -R $USER:$USER /var/www/word.diacritics.xyz

echo "<h1>Welcome to word.diacritics.xyz</h1>" > /var/www/word.diacritics.xyz/index.html
sudo ln -s /etc/nginx/sites-available/word.diacritics.xyz /etc/nginx/sites-enabled/
sudo nginx -t 
sudo systemctl reload nginx

sudo adduser $USER www-data
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R g+rw /var/www/html

curl -H "Host: word.diacritics.xyz" http://152.67.108.218

sudo apt install certbot python3-certbot-nginx

sudo certbot --nginx certonly -d word.diacritics.xyz -d www.word.diacritics.xyz

Certificate is saved at: /etc/letsencrypt/live/word.diacritics.xyz/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/word.diacritics.xyz/privkey.pem

mkdir word.diacritics.xyz
cd word.diacritics.xyz/
mkdir logs
mkdir public
cd ..
chmod -R 755 word.diacritics.xyz/

cd /etc/nginx/sites-available
sudo nano word.diacritics.xyz

```
server {
    listen 443 ssl;
    listen [::]:443 ssl http2;

    server_name word.diacritics.xyz;

    ssl_certificate /etc/letsencrypt/live/word.diacritics.xyz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/word.diacritics.xyz/privkey.pem;

    access_log /home/ubuntu/word.diacritics.xyz/logs/access.log;
    error_log /home/ubuntu/word.diacritics.xyz/logs/error.log;

    root /home/ubuntu/word.diacritics.xyz/public/;
    index index.php;

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

server {
    listen 443 ssl;
    listen [::]:443 ssl http2;

    server_name www.word.diacritics.xyz;

    ssl_certificate /etc/letsencrypt/live/word.diacritics.xyz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/word.diacritics.xyz/privkey.pem;

    return 301 https://word.diacritics.xyz$request_uri;
}

server {
    listen 80;
    listen [::]:80;

    server_name word.diacritics.xyz www.word.diacritics.xyz;

    return 301 https://word.diacritics.xyz$request_uri;
}

```

sudo ln -s /etc/nginx/sites-available/word.diacritics.xyz /etc/nginx/sites-enabled/word.diacritics.xyz

- Fix ownership
sudo chown ubuntu:www-data /home/ubuntu/word.diacritics.xyz/public/index.php

-  Fix permissions
sudo chmod 644 /home/ubuntu/word.diacritics.xyz/public/index.php

curl -H "Host: word.diacritics.xyz" https://localhost -k

sudo chown -R www-data:www-data /home/ubuntu/word.diacritics.xyz

sudo chmod -R 755 /home/ubuntu/word.diacritics.xyz
sudo chmod 644 /home/ubuntu/word.diacritics.xyz/public/index.php


curl -H "Host: word.diacritics.xyz" https://localhost -k

### mysql
CREATE DATABASE wp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_520_ci;
CREATE USER 'word'@'localhost' IDENTIFIED BY 'password';

GRANT ALL PRIVILEGES ON word.* TO 'word'@'localhost';
FLUSH PRIVILEGES;
exit;

wp core download

- Add ubuntu user to www-data group
sudo usermod -a -G www-data ubuntu

- Set group write permissions
sudo chmod -R g+w /home/ubuntu/word.diacritics.xyz

You'll need to log out and back in for the group change to take effect

wp core config --dbname=wp --dbuser=word --dbpass='password'

error establishing database
CREATE DATABASE IF NOT EXISTS wordpress_db;
CREATE USER IF NOT EXISTS 'wordpress_user'@'localhost' IDENTIFIED BY 'your_secure_password';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;



### best utilities
sudo ss -tlnp | grep :80
sudo lsof -i :80