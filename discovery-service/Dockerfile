FROM openjdk:8
ADD target/discovery-spring-boot.jar discovery-spring-boot.jar
EXPOSE 8761
ENTRYPOINT ["java", "-jar", "product-spring-boot.jar"]
CMD	java -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom -jar /config-spring-boot.jar