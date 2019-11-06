# This docker file should be in {backend-project-home}/Dockerfile
FROM openjdk:11.0.4-jdk

WORKDIR /app
COPY ./target/devday2019project-0.0.1-SNAPSHOT.jar ./

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app/devday2019project-0.0.1-SNAPSHOT.jar"]
