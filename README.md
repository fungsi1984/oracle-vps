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
