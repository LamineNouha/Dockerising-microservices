FROM openjdk:8
ADD target/discovery-spring-boot.jar discovery-spring-boot.jar
EXPOSE 8761
ENTRYPOINT ["java", "-jar", "discovery-spring-boot.jar"]
CMD	sleep 60  && \ java -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom -jar /discovery-spring-boot.jar