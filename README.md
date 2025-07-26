### some notes

```
ip tables in oracle doc not works, this temporary solution
sudo iptables -I INPUT 1 -p tcp --dport 80 -j ACCEPT
sudo netfilter-persistent save
```
  
