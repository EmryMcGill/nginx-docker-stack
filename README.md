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
    networks:
      - mynetwork
```

