FROM openjdk:8
ADD target/config-spring-boot.jar config-spring-boot.jar
EXPOSE 8888
ENTRYPOINT ["java", "-jar", "config-spring-boot.jar"]
CMD	java -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom -jar /config-spring-boot.jar