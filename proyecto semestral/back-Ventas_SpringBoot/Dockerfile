# Etapa 1: Construcción multi-stage 
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

# Etapa 2: Ejecución (Optimización y Seguridad)
FROM node:18-alpine
WORKDIR /app

# Crear usuario no root
RUN addgroup -S devopsgroup && adduser -S devopsuser -G devopsgroup
COPY --from=builder /app ./
USER devopsuser
EXPOSE 3000
CMD ["npm", "start"]