# Dockerizing-microservices
## Downloading Docker and starting the Docker engine
![screen shot 2017-12-19 at 11 02 48 pm](https://user-images.githubusercontent.com/25775136/34181318-18b00d08-e512-11e7-86b8-bdfca734d37e.png)
## Microservices to deploy on Docker
* Product Service: The main service, which offers a REST API listing all products.
* Config Service : Configuration service, whose role is to centralize the configuration files of the various microservices in a single place.
* Proxy Service: A gateway that handles the routing of a request to one of the instances of a service, in order to automatically manage the load distribution.
* Discovery Service: Service that records service instances for discovery by other services.

The resulting **architecture**:

![screen shot 2017-12-19 at 11 07 43 pm](https://user-images.githubusercontent.com/25775136/34181320-1c1c63b0-e512-11e7-8a11-1bde73eb8355.png)

## Dockerising our Microservices
So we have our microservices which we can run on localhost

![screen shot 2017-12-19 at 11 08 22 pm](https://user-images.githubusercontent.com/25775136/34181325-1f9b657c-e512-11e7-94ae-5ae8027757ae.png)

What we'd like to do is set up our microservices so that we can create  Docker Images from it, allowing us to deploy our services anywhere which supports docker.

## How to create a docker image from a spring boot application ?
Our first goal is to generate a Docker image from each Spring boot application using maven. This can be achieved by:

* Creating a Dockerfile. (method 1) <!-- on branch master -->
* The Spotify docker maven plugin. (method 2)<!-- on branch method2 -->

We worked with the two methods,the work with method1 is pushed on the **master branch** and the work witch method2 is pushed on **method2 branch**.

### Method 1: Creating a Dockerfile
For each microservice project we have to create a new text file called Dockerfile.

We will deal just with product-service project and you have to repeat the same thing with the rest of our microservices.

So, in the product-service project we create a Dockerfile with the content below:

```
FROM openjdk:8
ADD target/product-spring-boot.jar product-spring-boot.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "product-spring-boot.jar"]
CMD	java -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom -jar /config-spring-boot.jar
```

* **FROM**: It defines the base image to use to start the build process.
* **ADD**: command witch gets two arguments: a source and a destination. It basically copies the files from the source on the host into the container's own filesystem at the set destination.
We copied the jar in the root directory of the container.
* **EXPOSE**: Exposing this particular container on a port 8080.
* **ENTRYPOINT**: When a Docker container starts, it calls its entrypoint script, we had passed to it some arguments on how to run the entrypoint.
* **CMD**: tells docker that this image should run the command(s).

But before, we renamed the jar because it is huge by defining a finalName in pom.xml like this:

![screen shot 2017-12-19 at 11 08 45 pm](https://user-images.githubusercontent.com/25775136/34181330-23a409f8-e512-11e7-9efd-25e6e9b91c5e.png)

In order to create an **image** on witch we will deploy our jar, we have to **build our image first**:
```
$ docker build -f Dockerfile -t product-spring-boot .
```

![screen shot 2017-12-19 at 11 08 59 pm](https://user-images.githubusercontent.com/25775136/34181334-2781dab4-e512-11e7-81cc-8b1dec819f7d.png)

##### Checking images
```
$ docker images
```
![screen shot 2017-12-19 at 11 09 14 pm](https://user-images.githubusercontent.com/25775136/34181338-2b0f4edc-e512-11e7-9185-06e10ec45b2e.png)

We repeat the same thing with the rest of our microservices

![screen shot 2017-12-19 at 11 09 48 pm](https://user-images.githubusercontent.com/25775136/34181344-2e2901ee-e512-11e7-9fd0-0d3f3ac242e0.png)

##### Running images
```
$ docker run -p 8080:8080 product-spring-boot
```

a container is created and the image is up.
Now, doing the same thing with config-service, proxy-service, discovery-service.


### Method 2: Using the Spotify docker maven plugin
Our first goal is to generate a Docker image from a Spring boot application using maven. This can be achieved by the Spotify docker maven plugin. It eliminates the need to create a Dockerfile, instead all important properties like the base image and entrypoint can be set in the pom.xml like this:
```
<plugin>
				<groupId>com.spotify</groupId>
				<artifactId>docker-maven-plugin</artifactId>
				<version>0.4.10</version>

				<configuration>
					<imageName>${project.artifactId}</imageName>
					<baseImage>java:8</baseImage>
					<entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>
					<!-- copy the service's jar file from target into the root directory of the image -->
					<resources>
						<resource>
							<targetPath>/</targetPath>
							<directory>${project.build.directory}</directory>
							<include>${project.build.finalName}.jar</include>
						</resource>
					</resources>

				</configuration>
			</plugin>
```

And we edit our configuration in order to have this order of execution:

![screen shot 2017-12-19 at 11 10 01 pm](https://user-images.githubusercontent.com/25775136/34181352-31808f9c-e512-11e7-9e00-15afdc3652ef.png)

We repeat the same thing with all projects.

So, in the end we will have our images:
* Ones created with Dockerfiles
* Ones created with Spotify docker maven plugin 

##### Checking images

![screen shot 2017-12-19 at 11 10 17 pm](https://user-images.githubusercontent.com/25775136/34181360-342f7280-e512-11e7-94e3-4c5343fb9b66.png)

## Creating and running containers
* The first thing that we have to do is pushing the **config files** used by our micro services on GitHub in order to be accessible from docker.
https://github.com/LamineNouha/spring-boot-microservices-configfiles.git
* Then we launch the **config microservice**
```
$ docker run -it -p 8888:8888   config-spring-boot --spring.cloud.config.server.git.uri=https://github.com/LamineNouha/spring-boot-microservices-configfiles 
```

![screen shot 2017-12-19 at 11 10 35 pm](https://user-images.githubusercontent.com/25775136/34181371-385a03ac-e512-11e7-9cf4-ad3a8a19d20b.png)

* Now the config microservice is launched, we need to get his IP address as a container in order to launch other microsevices.
```
CONFIG_CONTAINER_ID=`docker ps -a | grep config-spring-boot | cut -d " " -f1`
```

```
CONFIG_IP_ADDRESS=`docker inspect -f "{{ .NetworkSettings.IPAddress }}" $CONFIG_CONTAINER_ID`
```

![screen shot 2017-12-19 at 11 10 46 pm](https://user-images.githubusercontent.com/25775136/34181375-3cb16922-e512-11e7-8bc7-6179be325e27.png)

* We launch the product microservice
```
$ docker run -it -p 8080:8080 product-spring-boot -e --spring.cloud.config.uri=http://$CONFIG_IP_ADDRESS:8888LamineNouha/spring-boot-microservices-configfiles 
```
![screen shot 2017-12-19 at 11 10 59 pm](https://user-images.githubusercontent.com/25775136/34181378-4072553a-e512-11e7-83b0-a7276572b1f8.png)

* We launch the discovery microservice
```
$ docker run -it -p 8761:8761 discovery-spring-boot \ -e --spring.cloud.config.uri=http://$CONFIG_IP_ADDRESS:8888
```
![screen shot 2017-12-19 at 11 11 17 pm](https://user-images.githubusercontent.com/25775136/34181385-43a1fce2-e512-11e7-933d-09a4df571e1b.png)

* And finally, We launch the proxy microservice
```
$ docker run -it -p 9999:9999 proxy-spring-boot
```
![screen shot 2017-12-19 at 11 11 28 pm](https://user-images.githubusercontent.com/25775136/34181390-47b7b182-e512-11e7-856d-d19d5ee69742.png)

We have to respect the **order** of running microserices, therefore we can:
* add to the Dockerfile of each microservice  a set of time that a microservice have to **wait** before executing the command defined by CMD.
For the discovery microservice for example:
```
CMD	sleep 30  && \ java -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom -jar /discovery-spring-boot.jar
```

* Or, use **docker-compose** to start up multiple containers and set up a container network.
```
version: '2.0'
services:
  config-spring-boot:
    image: config-spring-boot
    ports:
        - "8888:8888"
    networks:
        - seted-network

  product-spring-boot:
    image: product-spring-boot
    
    depends_on:
        - config-spring-boot
    networks:
        - seted-network

  discovery-spring-boot:
    image: discovery-spring-boot
    ports:
        - "8761:8761"
    depends_on:
        - config-spring-boot
    networks:
        - seted-network

  proxy-spring-boot:
    image: product-spring-boot
    ports:
        - "9999:9999"
    depends_on:
        - config-spring-boot
    networks:
        - seted-network

networks:
  seted-network:
    driver: bridge
```

The above example starts our 4 containers. The depends_on defines the startup order and how containers see the other. The file also defines the exposed ports of the services.
In order to launch microservices as described in the docker-compose.yml we execute:
```
$ docker-compose up -d
```

![screen shot 2017-12-19 at 11 11 43 pm](https://user-images.githubusercontent.com/25775136/34181393-4c52be3a-e512-11e7-803d-f54ffd13225a.png)

If we want create 3 instances of a product microservice we can use Docker easy scaling of the services like this:

##### Important:
The "product-spring-boot" service specifies a port on the host. If multiple containers for this service are created on a single host, the port will clash.
So we have modify the docker-compose.ylm by removing the port specification from product microservice like this:

![screen shot 2017-12-19 at 11 11 54 pm](https://user-images.githubusercontent.com/25775136/34181404-530360b8-e512-11e7-8bdf-4df4a03ba2f2.png)

So when we launch, the instance witch is already created is started and the 2 new other instances are created

![screen shot 2017-12-19 at 11 12 02 pm](https://user-images.githubusercontent.com/25775136/34181410-562beb2a-e512-11e7-9a9b-580a291bff79.png)

## References
https://www.youtube.com/watch?v=FlSup_eelYE&t=1104s

https://exampledriven.wordpress.com/2016/06/24/spring-boot-docker-example/

http://www.dwmkerr.com/learn-docker-by-building-a-microservice/
