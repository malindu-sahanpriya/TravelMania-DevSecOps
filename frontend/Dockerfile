# Stage 1: Build Vite React Static Assets
FROM node:20-alpine AS builder
WORKDIR /build

COPY package.json package-lock.json* ./
RUN npm install

COPY . .
RUN npm run build

# Stage 2: Hardened Unprivileged Nginx Runtime
FROM nginxinc/nginx-unprivileged:1.27-alpine AS runtime

USER root
RUN apk update && apk upgrade --no-cache && \
    rm -rf /var/cache/apk/*

COPY --chown=nginx:nginx nginx.conf /etc/nginx/conf.d/default.conf

# Vite outputs to /build/dist
COPY --from=builder --chown=nginx:nginx /build/dist /usr/share/nginx/html

USER 101:101
EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]