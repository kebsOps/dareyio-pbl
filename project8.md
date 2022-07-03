**LOAD BALANCER SOLUTION WITH APACHE**


<img width="832" alt="image" src="https://user-images.githubusercontent.com/10085348/177053642-13aba9db-b7a4-4a8d-a14e-08ab4694f0b9.png">


**Task**


Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 intance.

Make sure that users can be served by Web servers through the Load Balancer.

<img width="820" alt="image" src="https://user-images.githubusercontent.com/10085348/177053690-bb0637d1-673d-489b-b1fd-575828c86bea.png">




Configure Apache As A Load Balancer


Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb

<img width="1242" alt="apache lb instance" src="https://user-images.githubusercontent.com/10085348/177053759-0ca42cac-3aad-43c5-b69c-330842e0fce5.png">


Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.


<img width="1235" alt="open port 80 in Security groups" src="https://user-images.githubusercontent.com/10085348/177053781-1fbdbf26-178d-491b-93f1-27e45436f44a.png">


Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers


<img width="1104" alt="Install apache2" src="https://user-images.githubusercontent.com/10085348/177053804-0212b320-c4f1-40ce-9c7e-e9462205e6f5.png">

<img width="720" alt="Enable the following modules" src="https://user-images.githubusercontent.com/10085348/177053818-3a49fe69-befb-44bb-a5e3-55fbd1708701.png">

Configure load balancing

<img width="663" alt="Configure Load Balancer" src="https://user-images.githubusercontent.com/10085348/177053847-849afdac-fb57-4bdd-bbfa-01d8aa0e3551.png">


<img width="736" alt="Restart apache2 service and verify apache2 is running" src="https://user-images.githubusercontent.com/10085348/177053911-e6054240-f298-4175-a66b-efd287dd5edf.png">


Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:


<img width="1329" alt="LB working" src="https://user-images.githubusercontent.com/10085348/177054016-5cff1223-0ad1-4f56-9694-b964f1f59b5d.png">


Open two ssh/Putty consoles for both Web Servers and run following command:


sudo tail -f /var/log/httpd/access_log

Try to refresh your browser page several times and make sure that both servers receive HTTP GET requests from your LB
– new records must appear in each server’s log file. 
The number of requests to each server will be approximately the same since we set *loadfactor* to the same value for both servers
– it means that traffic will be distributed evenly between them.

<img width="1420" alt="HTTP GET requests from Load Balancer" src="https://user-images.githubusercontent.com/10085348/177053949-3b0ad19d-c6ad-4b6d-9b30-b6033372dcf3.png">




