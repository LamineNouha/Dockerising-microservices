FROM openjdk:8
ADD target/product-spring-boot.jar product-spring-boot.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "product-spring-boot.jar"]
CMD	sleep 30  && \ java -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom -jar /product-spring-boot.jar
