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

#install locale   --change with your locale information
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


