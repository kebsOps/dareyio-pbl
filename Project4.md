**MEAN STACK DEPLOYMENT TO UBUNTU IN AWS**

Run The following Commands to update and upgrade Ubunutu:

sudo apt update
sudo apt upgrade

![Ubuntu update](https://user-images.githubusercontent.com/10085348/156920155-b3cb1225-d951-4b3a-bd54-42c9d0f89682.png)
![Ubuntu upgrade](https://user-images.githubusercontent.com/10085348/156920156-e169d67a-6d75-400f-a07e-7c20d0c03780.png)

Add certificates

![Add Certificates](https://user-images.githubusercontent.com/10085348/156920216-970951c0-888d-416a-9838-4c1a9764ea02.png)
![Add Certificates 2](https://user-images.githubusercontent.com/10085348/156920219-53036ac7-17d7-44c3-818e-a3370da45b9c.png)


Run the command to install NodeJS
sudo apt install -y nodejs

![Nodejs installation](https://user-images.githubusercontent.com/10085348/156920300-2765b473-78f0-46dc-bbe1-a6c1845f0c17.png)

Install MongoDB

![Installing mongodb](https://user-images.githubusercontent.com/10085348/156920396-5fc2f505-fb6e-4cfe-9b2d-55f14234350b.png)

Start and verify that Mongo service is up

![Start and verify mongo service is up](https://user-images.githubusercontent.com/10085348/156920425-2714a0bd-f728-42d3-bd7d-be826b5b406d.png)


Install body-parser package

![Install body-parser](https://user-images.githubusercontent.com/10085348/156920487-1db4a6a2-d216-4e5a-bdb5-3cdc39c2ce9e.png)

Create a folder named ‘Books’, In the Books directory, Initialize npm 

![Create named folder named Books and initialize npm project](https://user-images.githubusercontent.com/10085348/156920521-000325ff-aa82-4499-8a0f-07a71eb106b0.png)

Create a file called server.js and insert code

![Create a file called server_js and insert code](https://user-images.githubusercontent.com/10085348/156920645-8e68548e-d69c-4a99-8c57-161dec2964de.png)

In 'Books' folder, create a folder named 'apps'

![In ‘Books’ folder, create a folder named apps](https://user-images.githubusercontent.com/10085348/156920807-bfb5a9a3-845b-4552-ad91-9f9d571ec392.png)

Create a route.js file and paste the code into the file in the /Books/apps folder

![Create a route_js file and paste the code into the file](https://user-images.githubusercontent.com/10085348/156920902-5da80974-bd85-41c3-b600-24778bc3d1cb.png)

In the ‘apps’ folder, create a folder named models, Create a file named book.js

![In the ‘apps’ folder, create a folder named models_Create a file named book js](https://user-images.githubusercontent.com/10085348/156920947-1c428903-8d5b-4221-ae13-0a6cf3739c48.png)

Navigate to Books Directory create public folder and script.js file

![Navigate to Books Directory create public folder and script_js file](https://user-images.githubusercontent.com/10085348/156921040-8d4345f2-dfd3-46e6-a2b1-7e74e77ee379.png)

Navigate to Books directory and start node server

![Start node server](https://user-images.githubusercontent.com/10085348/156921155-25e53213-ee0c-4483-83f3-50d31aac306d.png)

Open up TCP port 3300 in your AWS EC2 Instance

![Open TCP port 3300 in your AWS EC2 Instance](https://user-images.githubusercontent.com/10085348/156921208-2c5cb0b0-d5ad-467d-95ca-4b612bb376b8.png)

Web Book Register Application running on the web browser

![Web Book Register Application](https://user-images.githubusercontent.com/10085348/156921227-6fe10f5e-bce2-48af-b45c-0109e1b9afc5.png)
