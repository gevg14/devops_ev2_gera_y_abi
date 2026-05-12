# ─────────────────────────────────────────────
# Stage 1: Build – compila el JAR con Maven
# ─────────────────────────────────────────────
FROM maven:3.9.6-eclipse-temurin-17-alpine AS builder

WORKDIR /app

# Cache de dependencias Maven separado del código fuente
COPY Springboot-API-REST/pom.xml .
RUN mvn dependency:go-offline -B

# Compilar sin ejecutar tests
COPY Springboot-API-REST/src ./src
RUN mvn clean package -DskipTests -B

# ─────────────────────────────────────────────
# Stage 2: Runtime – imagen mínima de producción
# ─────────────────────────────────────────────
FROM eclipse-temurin:17-jre-alpine

LABEL maintainer="Innovatech Chile"
LABEL service="back-ventas"

# Crear usuario no-root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

RUN chown appuser:appgroup app.jar

USER appuser

EXPOSE 8080

ENV SPRING_PROFILES_ACTIVE=prod

ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-jar", "app.jar"]
