# ─────────────────────────────────────────────
# Stage 1: Build – compila el JAR con Maven
# ─────────────────────────────────────────────
FROM maven:3.9.6-eclipse-temurin-17-alpine AS builder

WORKDIR /app

# Copiamos primero solo el pom.xml para aprovechar el cache de capas
# Si el pom no cambia, Maven no re-descarga dependencias
COPY Springboot-API-REST-DESPACHO/pom.xml .
RUN mvn dependency:go-offline -B

# Copiamos el código fuente y compilamos (sin tests para agilizar el build)
COPY Springboot-API-REST-DESPACHO/src ./src
RUN mvn clean package -DskipTests -B

# ─────────────────────────────────────────────
# Stage 2: Runtime – imagen mínima de producción
# ─────────────────────────────────────────────
FROM eclipse-temurin:17-jre-alpine

# Metadatos de la imagen
LABEL maintainer="Innovatech Chile"
LABEL service="back-despachos"

# Crear usuario no-root para ejecutar la aplicación (seguridad)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copiar solo el JAR generado en el stage anterior
COPY --from=builder /app/target/*.jar app.jar

# Cambiar propietario al usuario no-root
RUN chown appuser:appgroup app.jar

# Cambiar al usuario no-root
USER appuser

# Puerto de la aplicación Spring Boot
EXPOSE 8081

# Variables de entorno con valores por defecto (se sobreescriben en docker-compose/EC2)
ENV SPRING_PROFILES_ACTIVE=prod

# Comando de arranque con opciones de JVM optimizadas para contenedor
ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-jar", "app.jar"]
