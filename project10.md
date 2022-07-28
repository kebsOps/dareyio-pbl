## PROJECT 10

## LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

<img width="956" alt="image" src="https://user-images.githubusercontent.com/10085348/181587706-12bfbda2-3601-4ab3-992a-d79e2651a75c.png">

Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)

<img width="1237" alt="image" src="https://user-images.githubusercontent.com/10085348/181602540-91f7f7fe-6cbb-4853-96ba-2c89e4b18a60.png">


Update `` /etc/hosts `` file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

<img width="509" alt="image" src="https://user-images.githubusercontent.com/10085348/181601948-4fd0b6ed-7ed2-49ef-85bf-be4538222c17.png">

Use the commands below to dpdate the instance and Install Nginx

```
sudo apt update
sudo apt install nginx
```

<img width="1021" alt="image" src="https://user-images.githubusercontent.com/10085348/181605126-0c3ef0ca-98a0-4fc5-9cdb-94afae095bab.png">

Open the default nginx configuration file
```
sudo vi /etc/nginx/nginx.conf
```
insert following configuration into http section

<img width="464" alt="image" src="https://user-images.githubusercontent.com/10085348/181606909-694bc925-7d0b-4d05-b263-0127a467f282.png">

Restart Nginx and make sure the service is up and running
```
sudo systemctl restart nginx
sudo systemctl status nginx
```

<img width="880" alt="image" src="https://user-images.githubusercontent.com/10085348/181607836-672aec9a-c4d7-468e-9cc3-7ec54b7c6536.png">

### REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES


