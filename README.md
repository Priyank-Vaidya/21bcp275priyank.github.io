# Docker File explaination

The starting of the dockerfile specifies the author name and the dockerfile version

`FROM node:18.16.0 `\
`FROM python:3.8-slim`

These are the image distributions along with their versions which will be used for executing our application inside the container. 

`WORKDIR /backend`

With this command we are setting the working directory as the backend, which basically specifies the reference directory and files from /backend directory level in the container file system.

`RUN apt-get update && apt-get upgrade -y && \` \
    `npm `

Command used in linux to update the system after package installation as done at the first step.

## First stage of installation

`COPY package*.json ./`\
`RUN npm install` 

Firstly this copies the server package.json dependancy file. We aim to first download all the necessary dependancies and then coping entire codebase to run the application in container

`COPY . .`

This copies all the files from the home directory to the container directory i.e /backend


## Second stage of installation and configuration

`WORKDIR /backend/ml_model`

Setting the working directory in the container file system 1 layer below of /backend for ml_model

`COPY ./ml_model/requirements.txt .`

First argument after COPY specifies the relative path of your local application and the one after it denotes the relative path with reference to the WORKDIR

Copying only the requirements first

This command will copy the contents of ml_model directory to the container /backend/ml_model directory

`RUN pip install -r requirements.txt `

Installing all the requirements for running the python application


`COPY ./ml_model ./backend/ml_model`

Copying all the code in the ml_model local directory to the container directory

`COPY ./configure-cloudsql-password.sh ../scripts`

In order to configure the password for cloudsql for storing the predictions of ml_model, I have written the bash file for performing and configuring the Google Cloud cloudsql.

This copies the bash file into the /scripts directory at the /backend level

`ENV GOOGLE_CLOUD_PROJECT ${GOOGLE_CLOUD_PROJECT}` \
`ENV GOOGLE_CLOUD_SQL_USERNAME ${GOOGLE_CLOUD_SQL_USERNAME}`\
`ENV GOOGLE_CLOUD_SQL_PASSWORD ${GOOGLE_CLOUD_SQL_PASSWORD}`\
`ENV GOOGLE_CLOUD_SQL_DATABASE ${GOOGLE_CLOUD_SQL_DATABASE}`\
`ENV GOOGLE_CLOUD_SQL_HOST ${GOOGLE_CLOUD_SQL_HOST}`\
`ENV GOOGLE_CLOUD_SQL_PORT ${GOOGLE_CLOUD_SQL_PORT}`


As you see that environments specified after {$} needs to be configured by the automated workflow while initializing the VM on which  the application needs to be run. 

Here VM is an abbreviation of linux system on which the containers are executed 

`RUN python3 db.py`

Running the db.py file for using the cloudsql 

`EXPOSE 3000`

Exposing container port 3000 for open traffic to access the application

`CMD ["python3", "stock_market_prediction.py"]`

Start executing the predictions by running the python scripts

`CMD ["npm", "start"]`

Finally start the application by running it in node environment



# Docker-compose.yml

There are 3 services for running interdependent containers \
db \
app \
psql \

### db 
It uses the postgres public image from docker registery and run the necessary configuration set up from configuration-postgres.sh bash file 

There are some other parameters eg data path, volume, bridge

### app

Uses the dockerfile as it's dependancy on executing application
it depends on db services
is connected to same network as the db i.e bridge

### psql 

depends on db service
connected to same network for execution
 
