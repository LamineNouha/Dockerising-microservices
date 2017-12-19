FROM openjdk:8
ADD target/proxy-spring-boot.jar proxy-spring-boot.jar
EXPOSE 9999
ENTRYPOINT ["java", "-jar", "proxy-spring-boot.jar"]
CMD	sleep 100  && \ java -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/./urandom -jar /proxy-spring-boot.jar