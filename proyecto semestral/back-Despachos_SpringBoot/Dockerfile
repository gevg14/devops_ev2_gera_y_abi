FROM maven:3.9.6-eclipse-temurin-17 AS builder
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests
FROM eclipse-temurin:17-jdk-alpine
WORKDIR /app
# Usuario no root
RUN addgroup -S springgroup && adduser -S springuser -G springgroup
COPY --from=builder /app/target/*.jar app.jar
USER springuser
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]