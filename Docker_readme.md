# <center> Docker
### Why docker
If you want to deploy an application that uses different services, and each service has different requirements, deploying the application can be very challenging.   
With docker you can run each component on a separate container, with its own dependencies and libraries, all in the same virtual machine.   
A **Containers**: isolated environments. They can have their own processor, network, and mounts. They all share the same OS Kernel.
A **OS** is made by: **OS Kernel** (it interacts with the HW, similar across many OS), and a bunch of **Softwares** (What really define a specific OS).
This means you can run docker across all OS that share the same OS Kernel. You can still run a linux docker container on a Windows machine, but actually you are running
the linux docker container on a linux virtual machin on Windows.   
**Containers vs virtual machines**: the VM has multiple OS on the same hardware, and this make consume more resources. Docker is much lighter and faster.
For complex systmes you can have VM and docker on them together.  
Two importamt concepts:
* Docker Image: a package with a template.
* Docker container: a running instance of an Image. You can have many containers runnign for a single Image.  
NOTE: many products have been dockerized already and are publicly available, and you do not need to re-create them. Simply run ```docker run mongodb``` to run an istance of mongodb.
You can also create your own Image and make it publit to everyone.  
Docker make simple the interaction between Development and Operation, contributing to Dev/Ops culture.

### Basic Commands
* Run a container associated to an Image
  * ```docker run -d <image name>```
  * ```docker run -d <image name>:<tag>``` (if there is no tag, it will pull the latest)
  * If the Image is not present it will pull down the Image from docker-hub.
  * The option -d allow you to run detached. So it runs in background, and you can keep working. Otherwise you see the container running and with ctr-c you quit it.
* Run and enter interactively a container
  * ```docker run -it centos bash``` (you need a command like bash to keep it active, otherwise it will run and exit immediatly)  
  * ```docker run -it my_app``` (my_app ask my name, and reply with a message. I need to use -i=interactive (listen for input) and -t=terminal (it is attached to the terminal).
  * ```docker run -d centos sleep 20``` (it run detached for 20 seconds)
* Dowload an Image without running a container associated with it
  * ```docker pull <image name>```
* List all running containers
  * ```docker ps```
* List all containers, even if stopped:
  * ```docker ps -a```
* Stop a running container
  * ```docker stop <container name>```
* Remove a container permanentily
  * ```docker rm <container name>```
  * Now it will not be visible even with docker ps -a.
* List all available Images
  * ```docker images```
* Remove an Image
  * ```docker rmi <image name>```
  * To be able to do so, you need first to remve all containers using that Image.  
* Execute a command inside a container
  * ```docker exec <container id> cat file.txt```   
* Attach to a docker container running in backgroud   
  * ```docker attach <docker id>```
* Inspect all deatails of a container
  * ```docker inspect <container name>```
* See thge Log of a container (std out of a container)
  * ```docker logs <container name>```

#### Notes on the commands
* A container are not meant to host operating systems, but only applications (like databases, or programs). If you run 'docker run ubuntu' it will execute it and exit immediately.
The container only live as long as the program inside lives. Ubuntu in this case is used as base image for other applications.  
* the image name is by default looked into: 'docker.io/<username>/<image_repo>'. The 'docker.io' can be omitted, and if you only say 'docker run my-app' he will assume the image is in
'docker.io/my-app/my-app'
* Before pulling of pushing froma private registry you need to login: ```docker login private-registry.io```.  

### Web Application
Running a web application with 'docker run webapp' will make it run on a standard ip (address) and port. The ip is local to the docker host, so you need to open a browser within the
docker host.  
An external user can only access the docker host ip, so you need to map a port of the docker host ip, to the application ip and port.  
This can be done with ```docker run -p 80:5000 webapp``` (you are running port 80 on the docker host to the port of the app, 5000 in this case). You can this for multiple apps.

### Persistency
If you have a container hosting a mysql database, it has his own filesystem. You can make changes and create entry in the db, but once you stop the container and remove
(docker stop mysql; docker rm mysql) it is all gone. 
You can map a directory outside the container on the docker host to a directory insider the container:  
```docker run -v /opt/datadir:/var/lib/mysql mysql```
This way when mysql runs it will mount the direcotry and the directory will remain.

### Create your own Image
If you wnat to create an Image for your application (in this example Flask) you need to know:     
* The OS (e.g. Ubuntu)  
* Uldate the apt repo (for linux)  
* Install the dependencies (the apt command for linux)  
* Install the pyhton dependecies with pip  
* Copy the course code in /opt  
* Run the web server suing the Flask cmmand  
All this is in a **Docker file** that will have:  
```
FROM Ubuntu
RUN apt-get update && apt-get -y install python
RUN pip install flask-mysql
COPY . /opt/source-code  # Here you are copying the code like app.py, that are the app to dockerize.
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```
And the run: ```docker build . -f Dockerfile -t lpernie/my-custom-app```. 
Thsi will be available to your system only. To make it available to everyone: ```docker push lpernie/my-custom-app```.

#### Environmental Variables
If the app.py that you are dockerizing has a paramter par=32, and you want to change it you can replace it with ```par=os.environ.get('PAR')```.
Then you can create a container doing: ```docker run -e PAR=32 app.py``` and change that paramter easily.
You can see all the ENV on a running container with:   
```docker inspect <ontainer name>```  

#### Command vs Entry Point
The Docker File can have a line that says:
```CMD bash```
This tell that once you run the container this is the defaulkt command to execute. If the app is Ubuntu you are launching the shell, but Ubuntu sees no shell and quit.  
You can overwrite the CMD in the Docker File giving explicitely a command when running (docker run Ubuntu sleep 5).   
You can also place in the Docker file:
```ENTRYPOINT sleep```
and later run with docker run Ubuntu 5, and the 5 will be applied to 5 by default.
You can also use:
```
ENTRYPOINT sleep
CMD 5
```
and the defuakt value will be sleep 5, bu you can overwrite the 5 if you run 'docker run Ubuntu 10'.
NOTE: You can also overwrite the entrypoint at run time with:  
```docker run --entrypoint sleep_v2 Ubuntu 7```

### Docker Compose
If you have a complex project with many containers, you can create a **docker-compose.yml** file:
```
services:
  web:
    image: 'image1_app'
  database:
    image: 'image2_app'
```
and the run: ```docker-compose up```, but this only work for containers running on a single docker host.
Example: You vote across 2 options. The vote is collected by a DB, then it is processed, then it is store in another DB, then is made available.
You need to run:
```
docker run -d --name=redis redis # First DB. The --name will make this container availble with thsi name
docker run -d --name=db          # Second DB
docker run -d --name=vote -p 5000:80  --link redis:redis voting-app  # --link tells the voting app where to take the input DB (you map the redis container to the redis input expected by this container)
docker run -d --name=result --link db:db -p 5001:81 result-app  # Same here
docker run -d --name=worker worker --link db:db --link redis:redis  # Worker needs both DB, so you need to link them both.
```
To place everything into a **docker-compose.yml** file in this case you should write:
```
version: 3  # Tells Docker what version of docker-compose you are using, so it known how to read it. 
services:
  redis:
    image: redis
    networks:
      - back-end
  db:
    image: postdegres:9.4
    environment:  # This let you define environmental variabled used by the container.
      POSTGRES_USER: abc  
      POSTGRES_PASSWORD: abc
    networks:
      - back-end
  vote:
    image: voting-app
    ports:
      - 5000:80
    links:
      - redis:redis
    networks:
      - back-end
  result:
    image: result-app
    ports:
      - 5001:81
    links:
      - db:db
    networks:
      - front-end
      - back-end
  worker:
    image: worker
    links:
      - redis:redis
      - db:db
    networks:
      - front-end
      - back-end
networks:  # This is usefult if you want to use different networks for different part of the application.
  front-end
  back-end
```
And then: ```docker-compose up```.   
NOTE: if my application 'voting-app' has not an image ready to run, you need to build one. In this case you need change the line ```image: result-app``` with ```build: ./result```, where
that folder contains all in ingredient to build the image (Dockerfile, result.py, requirement.txt).

### Kubernetes Introduction
If you have many clients you need to run multiple time an instance of the same container (you have to do many 'docker run'), and you have to keep a close look on that.  
Also, if a host is down, all the docker will fail. This is a full time jobs, unless you have a **Container Orchestartion**. This allow you to instanctiate many containers and monitor
them efficiently. There are many options: Docker Swarm (You need to have many hosts, and a Swarm manager), Kubernets (most common, a bit harder), Mesos.  

