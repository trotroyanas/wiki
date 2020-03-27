You can use Docker to run FoundryVTT, you may use two different approach.

## A simple Dockerfile

You can check at [https://github.com/mikysan/simple-fvtt-dockerfile](https://github.com/mikysan/simple-fvtt-dockerfile).
You'll simply need to copy or download the **Dockerfile** you'll find in the repository into your fvtt directory and follow the instructions.

## Docker compose

### For this you only need 3 files.
<!--ts-->
      * Dockerfile
      * docker-compose.yml
      * foundryvtt-x.x.x.zip (Give by Patreon)

**Dockerfile** describe installation of your container

```yaml
FROM ubuntu:latest

# make directory
RUN mkdir -p /home/foundry/data/Config
RUN mkdir -p /home/foundry/app

# Set the foundry install home
ENV FOUNDRY_HOME=/home/foundry/app
ENV FOUNDRY_DATA=/home/foundry/data
# add a foundry group with a guid listed above

#langage
ENV LANG fr_FR.utf8

# Set the current working directory
WORKDIR "${FOUNDRY_HOME}"

# create the foundry use
RUN groupadd foundry
RUN useradd foundry -g foundry -b "${FOUNDRY_HOME}"

# installing unzip and bash shell
RUN apt-get update

#install locale --with your locate information
RUN ln -fs /usr/share/zoneinfo/Europe/Paris /etc/localtime
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get install -y tzdata
RUN dpkg-reconfigure --frontend noninteractive tzdata


# curl nodejs nginx unzip
RUN apt-get install apt-utils unzip curl -y
RUN curl -sL https://deb.nodesource.com/setup_12.x | bash -
RUN apt-get -y install openssl
RUN apt-get -y install nodejs
RUN apt-get -y install nginx


# Certificate
# run this command for generate Key & cert
# RUN openssl req -newkey rsa:4096 -x509 -sha256 -days 3650 -nodes -out "${FOUNDRY_DATA}/Config/foundry.crt" -keyout "${FOUNDRY_DATA}/Config/foundry.key" -subj "/C=FR/ST=Paris/L=Paris/O=Global Security/OU=>
# Copy Key & Cert here
# Add in /home/foundry/data/Config
# change options.json and add  Key & cert
#  "sslCert": "foundry.crt",
#  "sslKey": "foundry.key",


# nginx pararm
RUN rm /etc/nginx/sites-enabled/default
RUN service nginx restart

# copy found
COPY ./foundryvtt*.zip .

# unzip
RUN unzip foundryvtt*.zip
RUN rm foundryvtt*.zip

# clean
RUN apt-get autoremove
RUN apt-get clean

# Expose port
EXPOSE 30000
CMD [ "node", "resources/app/main.js", "--dataPath=/home/foundry/data" ]
```

###  **docker-compose.yml** describe creation image
```yaml
version: '3.3'

services:
   fvtt:
        build:
            context: ./
            dockerfile: ./Dockerfile
            args:
                UID: 1026
                GUID: 65534
        image: fvtt:1.11.0            
        container_name: FoudryVTT
        ports:
            - "30000:30000"
        restart: always

```

#### **foundryvtt-x.x.x.zip** FoundryVTT (Give by Patreon)
copy your foundryvtt-x.x.x.zip file to the same level as the other files


### *Build image*
`sudo docker-compose build -t fvtt:1.22.0 .`

After build image you can use or test your container with this commands :

Run the container (for test no save modifications)
#### `sudo docker run --restart=always --name FoundryVTT.x.x.x -p 30000:30000 -d fvtt:1.22.0`


#### Run the docker with volume map for save yours modifications : ####
```
sudo docker run --restart=always --name FoundryVTT.x.x.x -p 30000:30000 \
 -v /YourLocalDirectory1:/home/foundry/app \
 -v /yourLocalDirectory2:/home/foundry/data \
-d fvtt:1.22.0
```

