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
Insert following configuration into http section

<img width="464" alt="image" src="https://user-images.githubusercontent.com/10085348/181606909-694bc925-7d0b-4d05-b263-0127a467f282.png">

Restart Nginx and make sure the service is up and running
```
sudo systemctl restart nginx
sudo systemctl status nginx
```

<img width="880" alt="image" src="https://user-images.githubusercontent.com/10085348/181607836-672aec9a-c4d7-468e-9cc3-7ec54b7c6536.png">

### REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

Purchased a domain (toolingkb.xyz) from godaddy.com at $2.33

We goto Route 53 to Use the purchased domain

Create an hosted zone using purchased domain

<img width="1087" alt="image" src="https://user-images.githubusercontent.com/10085348/181878204-8decc177-5f64-4ef8-b02b-cebb558ddfeb.png">

Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

<img width="1235" alt="image" src="https://user-images.githubusercontent.com/10085348/181878512-8c9cbdb3-5658-464e-a414-5e8401d82abd.png">

<img width="821" alt="image" src="https://user-images.githubusercontent.com/10085348/181878632-8a8d2e90-c2a6-4c1a-a0d1-6b6d905beadd.png">

Update A record in your registrar to point to Nginx LB using Elastic IP address

<img width="1016" alt="image" src="https://user-images.githubusercontent.com/10085348/181879132-efe039a9-f3d4-4de6-888e-5e7c30f0d9fc.png">

<img width="1030" alt="image" src="https://user-images.githubusercontent.com/10085348/181879310-2a7b9e85-2a20-4575-aca2-e4a55e06a7ab.png">

<img width="990" alt="image" src="https://user-images.githubusercontent.com/10085348/181879399-42a1360a-2409-46b4-91bf-6e294aa582fc.png">

Configure Nginx to recognize your new domain name
Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com
i.e from www.domain.com to www.toolingkb.xyz)

<img width="382" alt="update server address" src="https://user-images.githubusercontent.com/10085348/181880001-96ebd809-becf-4a6c-a51f-d0ec08380fa6.png">

Make sure snapd service is active and running

<img width="1030" alt="image" src="https://user-images.githubusercontent.com/10085348/181880195-0c983272-fd7d-4cf6-bbdb-537daf28b142.png">

Install certbot

<img width="459" alt="image" src="https://user-images.githubusercontent.com/10085348/181880251-38204734-a5c8-49cd-a2da-cc48da167616.png">

Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it 

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

<img width="1469" alt="image" src="https://user-images.githubusercontent.com/10085348/182354158-927ed520-891e-4607-9bf3-151ec0d46501.png">

Set up periodical renewal of your SSL/TLS certificate
By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

You can test renewal command in dry-run mode

```
sudo certbot renew --dry-run
```
<img width="622" alt="image" src="https://user-images.githubusercontent.com/10085348/182356674-bc4d42bc-297d-4f1f-be0a-bb3f3eab5f66.png">

Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.

To do so, lets edit the crontab file with the following command:

```
crontab -e
```

Add following line:

```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

<img width="1033" alt="image" src="https://user-images.githubusercontent.com/10085348/182355659-f40b2fdd-dcb5-4fe1-b794-6042b3fe8107.png">


You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.


