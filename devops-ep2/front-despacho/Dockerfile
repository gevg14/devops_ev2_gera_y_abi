# ─────────────────────────────────────────────
# Stage 1: Build – genera los archivos estáticos con Vite
# ─────────────────────────────────────────────
FROM node:20-alpine AS builder

WORKDIR /app

# Copiar manifiestos primero para cache eficiente de dependencias
COPY front_despacho/package*.json ./
RUN npm ci --frozen-lockfile

# Copiar el resto del código y compilar
COPY front_despacho/ .
RUN npm run build

# ─────────────────────────────────────────────
# Stage 2: Runtime – Nginx sirve los estáticos
# ─────────────────────────────────────────────
FROM nginx:1.27-alpine

LABEL maintainer="Innovatech Chile"
LABEL service="front-despacho"

# Crear usuario no-root para Nginx
RUN addgroup -S nginxgroup && adduser -S nginxuser -G nginxgroup

# Copiar build de Vite al directorio de Nginx
COPY --from=builder /app/dist /usr/share/nginx/html

# Copiar configuración personalizada de Nginx
# (maneja SPA routing y proxy hacia los backends)
COPY front_despacho/nginx.conf /etc/nginx/conf.d/default.conf

# Ajustar permisos
RUN chown -R nginxuser:nginxgroup /usr/share/nginx/html && \
    chown -R nginxuser:nginxgroup /var/cache/nginx && \
    chown -R nginxuser:nginxgroup /var/log/nginx && \
    touch /var/run/nginx.pid && \
    chown -R nginxuser:nginxgroup /var/run/nginx.pid

USER nginxuser

# Solo el frontend es accesible desde Internet
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
