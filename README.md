# nginx-docker-stack
To hold my knoledge of configuring a Nginx reverse proxy with Docker containers to serve my apps to the internet

## Docker commands
- stop running containers: `docker compose down`
- start all container: `docker compose up --build -d`
- force a new build: `docker compose build <container name>`



## Project Setup

### 1. **Prepare Your Server**

You will need a server to run the reverse proxy and host your services. You can use:
- A **VPS** (e.g., from Linode, DigitalOcean, etc.)
- An **old laptop** repurposed as a server
- Any other server of your choice

For this guide, I'll be using an **Ubuntu instance** from Linode, but the steps will work for any server setup.

### 2. **Install Docker**
This will install the docker engine and the docker CLI, it will also automatically install the `docker compose plugin`

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli
```

### 3. **Folder Structure**
In my home directory I created a directory for the reverse-proxy and docker logic called `reverse-proxy` and another directory for my apps called `apps`
```bash
mkdir reverse-proxy
mkdir apps
```

### 3. **Docker Compose**
Docker Compose is a way to define a bunch of containers at once and let them talk to eachother. This is good for us since we want an easy way to manage all of our apps.
First we make a docker-compose.yml file inside the `reverse-proxy` folder to define all of the docker conainers for the proxy, and all the services we want to host. Here is the basic outline we need:
```
version: '3.8'

services:

networks:
  mynetwork:
    driver: bridge

volumes:
  pb_data:
```
Explination:
- `version` is simply the version of docker-compose format
- `services` this is where all of our docker containers will be defined
- `networks` here we create a network `mynetwork` so that all of our containers can talk to eachother
- `volumes` here we create any persistant storage we might need. I made `pb_data` since I will be using pocketbase for the database for most of my apps.

#### Nginx Container
First we will create the Nginx container. All taffic from the internet will run through this container.
```
services:
  nginx:
    image: nginx:latest
    container_name: nginx_proxy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ~/reverse-proxy/nginx.conf:/etc/nginx/nginx.conf:ro
      - /etc/letsencrypt:/etc/ssl:ro
    networks:
      - mynetwork
```
#### App Container
Now we can add a container for our app. Add this as another service. This is an example of my portfolio website made with react but your app might look a bit different
```
portfolio:
    build:
      context: ~/apps/portfolio/
      dockerfile: Dockerfile
      args:
        - VITE_PB_URI=h<link to api>
    container_name: portfolio
    restart: unless-stopped
    expose:
      - "4000"
    networks:
      - mynetwork
```
Explination:
- `portfolio` this is the name of our service.
- `build` since I am building my own image I need to specify where the Dockerfile is and the directory of the app.
- `container_name` this is the name of the docker container
- `expose` this exposes the specified port to the other containers on the same network (mynetwork)
- `networks` this says what network the container is in (make sure its the same as the nginx one)

### 4. **Nginx Config**
Now we need to create a nginx.conf (also in the reverse-proxy folder) file to configure the reverse-proxy. This conf file will be used by the Nginx container to direct traffic to our app based on the domain.
```
events {}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

}
```
Explination:
- `events` we won't use this but its required to be there.
- `http` this is where we will define the server block for our apps

#### adding a server block for our app
```
server {
        listen 80;
        server_name emrymcgill.com www.emrymcgill.com;

        return 301 https://emrymcgill.com$request_uri;
}
server {
    listen 443 ssl;
    server_name emrymcgill.com;

    ssl_certificate /etc/ssl/live/emrymcgill.com/fullchain.pem;
    ssl_certificate_key /etc/ssl/live/emrymcgill.com/privkey.pem;

    location / {
        proxy_pass http://portfolio:4000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Here we are adding 2 server blocks. The first one will redirect any http request to a https request.

### 5. **Adding SSL**
To add SSL we will use certbot and letsencrypt. First make sure they are both installed, then run this command to get certs: `sudo certbot certonly --manual --preferred-challenges dns` then follow the instructions to get the certs. Since we used the --manual flag, we cannot do automatic cert renewal, so before the certs expire just run that command again to renew them.
