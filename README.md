# nginx-docker-stack
To hold my knoledge of configuring a Nginx reverse proxy with Docker containers to serve my apps to the internet

## Docker commands
- stop running containers: `docker-compose down`
- start all container: `docker-compose up --build -d`
- force a new build: `docker-compose build <container name>`


## Docker-Compose
First we must make a docker-compose.yml file to define all of the docker conainers for the proxy, and all the services we want to host. Here is the basic outline we need
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

### Nginx Container
First we will create the Nginx container. All taffic from the internet will run through this container.

