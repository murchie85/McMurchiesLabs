# DOCKER - OVERVIEW 

![Docker](https://3kllhk1ibq34qk6sp3bhtox1-wpengine.netdna-ssl.com/wp-content/uploads/2017/01/how-to-deploy-java-apps-with-docker-a-quick-tutorial@3x-1560x760.png)  

## TOPICS COVERED 

1. [HIGH-LEVEL-STEPS](#HIGH-LEVEL-STEPS)
2. [DOCKER-BUILD-COMMANDS](#DOCKER-BUILD-COMMANDS)
3. [DOCKER-INFORMATION-COMMANDS](#DOCKER-INFORMATION-COMMANDS)
4. [DOCKER-CLEANUP-COMMANDS](#DOCKER-CLEANUP-COMMANDS)
5. [ON-BUILD-FLOW](#ON-BUILD-FLOW)
6. [DATA-CONTAINER](#DATA-CONTAINER)
7. [EXAMPLE-DOCKERFILE-CONFIG](#EXAMPLE-DOCKERFILE-CONFIG)

## HIGH-LEVEL-STEPS

With examples...

1. Create your app *(put it in src file normally)*
2. Write your docker file 
3. Build Image **(t allows you to add a friendly image name)** *DON'T* forget the period at the end
```
docker build -t MYIMAGENAME .
```
4. Run Image as container
```
docker run -d MYIMAGENAME
```
5. Optionally add volume flag `-v MYPATH/:/var/www/html/`  
  
## EXAMPLE

An example Python app 

1. Create a folder 
2. cd to to folder - add docker file like this one. **It adds everything in current directory to /code folder in container** 

```
FROM python:3
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
```

3. Build it by running `docker build -t MYIMAGENAME . ` **don't forget the period .**
4. `docker run --name charmander -d -p 8885:80 -v $pwd/:/opt madeup-image-name` This will create a container named charmander, in background `d` on external port 8885, current directory will be synced to opt folder
5. SSH to container with `docker exec -it charmander bash`  
  
  


# DOCKER-BUILD-COMMANDS 

### BUILD 


```
docker build -t MY_GIVEN_IMAGENAME .
```
  
### BUILD WITH NO CACHE 

```
docker build  --no-cache=true
```


# DOCKER RUN COMMANDS 


### RUN IN BACKGROUND 

USE D FLAG

```
docker run -d application-name
```
  



### RUN DEFINE NAME & PORT


```
docker run -d --name redisHostPort -p external:internal myappname:latest
```


Redis actually runs on 6379

```
docker run -d --name redisHostPort -p 6379:6379 redis:latest
```


*You can specify a particular IP address when you define the port mapping, for example, -p 127.0.0.1:6379:6379*


  

### RUN ON RANDOM AVAILABLE PORT 

```
docker run -d --name redisDynamic -p 6379 redis:latest
```
  


### RUN AND BIND VOLUME 

MYPATH=pwd + src

```
docker run -p 8880:80 -v MYPATH/:/var/www/html/ IMAGE-NAME-YOU-BUILT
```

**USE CURRENT DIRECTORY**

```
docker run -p 8880:80 -v $pwd/:/var/www/html/ IMAGE-NAME-YOU-BUILT
```



EXAMPLE
  

### RUN IN BACKGROUND WITH PORT, VOLUME AND NAME SPECIFIED 

```
docker run --name pikachu -d -p 8800:80 -v /Users/adammcmurchie/projects/docker/simple-apache-php/src:/var/www/html/ hello-docker
```

example 

```
docker run --name pikachu -d -p external:internal -v FULLPATHTOSRC:/var/www/html/ APP-NAME
```
  



### RUN AND LOGON AS BASH


```
docker run -it my-app bash
```

  

## ENVIRONMENT VARIABLES 


Using -e option, you can set the name and value as -e *MY_ENV=production*



```
docker run -d --name my-production-running-app -e NODE_ENV=production -p 3000:3000 my-nodejs-app

```


# DOCKER-INFORMATION-COMMANDS 


### GET CMD NAME

```
docker top NAME-U-ASSIGNED
```

### GET PID & CAT ENV

```
DBPID=$(pgrep CMD-NAME)
echo Service is $DBPID
ls /proc

cat /proc/$DBPID/environ
```



### FIND THE PORT

```
docker port redisDynamic 6379
```



### LIST RUNNING CONTAINERS

```
docker ps
```



### INSPECT 


```

docker run -d --name pikachu redis

docker inspect pikachu 

```


### Container Information

```
docker inspect <friendly-name|container-id>
```


### Container logs 

WONT WORK WITH IMAGE NAME  (Give it a --name when running)

```
docker logs <friendly-name|container-id>
```



# DOCKER-CLEANUP-COMMANDS

- first remove all stopped containers
- then remove images
- then clean up 

### Clean up containers 


First list them 
```
docker ps -a
```

Stop them all
```
docker stop $(docker ps -a -q)
```



Now clean up

```
docker rm $(docker ps -a -f status=exited -q)
```
  

Remove remaining manually

```
docker rm ID_or_Name ID_or_Name
```


### CLEAN UP IMAGES

LIST all images

```
docker images -a
``` 
  
REMOVE all in bulk

```
docker rmi $(docker images -a -q)
```

REMOVE remaining manually 

```
docker rmi Image Image

```

or if you can't remove them all then need to prune 

PRUNE 

```
docker system prune -a
```



# ON-BUILD-FLOW

1. Create a application dockerfile that references a ready to go image with `onbuild`
2. The ready to go image has ONBUILD commands which only run when the application image build is happening 

The Maven container is designed to compile java programs. Magically all your project's Dockerfile needs to do is reference the base container containing the ONBUILD intructions:

```
FROM maven:3.3-jdk-8-onbuild
CMD ["java","-jar","/usr/src/app/target/demo-1.0-SNAPSHOT-jar-with-dependencies.jar"]
```

The base image's Dockerfile tells all

```
FROM maven:3-jdk-8

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

ONBUILD ADD . /usr/src/app

ONBUILD RUN mvn install
```

There's a base image that has both Java and Maven installed and a series of instructions to copy files and run Maven.

The following answer gives a Java example

[How to build a docker container for a java app](https://stackoverflow.com/questions/31696439/how-to-build-a-docker-container-for-a-java-app/31710204#31710204)

  

# DATA-CONTAINER

Data Containers are containers whose sole responsibility is to be a place to store/manage data.

However, they don't run when you perform a `docker ps` command.

Lets use a well-known image called busybox, Data containers can be moved to other hosts and shipped. 

When creating the container, we also provide a -v option to define where other containers will be reading/saving data.

```
docker create -v /config --name adamsdataContainer busybox
```


This will pull busybox container down.

With the container in place, we can now copy files from our local client directory into the container.


```
echo 'configuration' >> config.conf


docker cp config.conf adamsdataContainer:/config/
```

Now our Data Container has our config, when we create other containers (like ubuntu below) we can pull in the volume from our adamsdataContainer.



Using the `--volumes-from <container> ` 


```
docker run --volumes-from adamsdataContainer ubuntu ls /config
```

`ls /config` will show us the config file we pased in.

If the volume existed, it would override, and we can map multiple volumes to an instance. 

## EXPORTING OUR DATACONTAINER 

```
docker export adamsdataContainer > dataContainer.tar
```

Now we can import it back using the following 


```
docker import dataContainer.tar
```


# GENERAL-TIPS

## Ignore 

To ignore say a paswords file just add a .dockerignore file 

```
echo passwords.txt >> .dockerignore
```
  
  It's wise to ignore .git directories


## SSH TO CONTAINER OR RUN COMMANDS 


To SSH run the following 

```
docker exec -it <container name> /bin/bash 
```

It may not allow bash so you can still run commands this way 

```
docker exec -it <container name> <command>
```


# EXAMPLE-DOCKERFILE-CONFIG


## NGINX 

Need a src/index 

```
#PORT SETUP AND COMMAND READY TO GO
FROM nginx:1.11-alpine
COPY src/index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```



## NODE JS 

```

# This is your Editor pane. Write the Dockerfile here and 
# use the command line to execute commands
FROM node:10-alpine

RUN mkdir -p /src/app

WORKDIR /src/app

COPY package.json /src/app/package.json

RUN npm install

COPY . /src/app

EXPOSE 3000

CMD [ "npm", "start" ]

```