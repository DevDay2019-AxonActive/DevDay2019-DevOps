FROM adoptopenjdk/maven-openjdk11 as build-stage
WORKDIR /app
COPY . ./
RUN mvn clean install -P initData # init data mode

FROM openjdk:11.0.4-jdk
WORKDIR /app
COPY --from=build-stage /app/target/devday2019project-0.0.1-SNAPSHOT.jar /app

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app/devday2019project-0.0.1-SNAPSHOT.jar"]
