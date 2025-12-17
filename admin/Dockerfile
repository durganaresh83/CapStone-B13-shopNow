############################
# Stage 1: Build (React)
############################
FROM node:20-bookworm-slim AS builder
WORKDIR /app

# Install build deps only in builder
RUN apt-get update && \
    apt-get install -y python3 make g++ && \
    rm -rf /var/lib/apt/lists/*

# Copy dependency files
COPY package.json package-lock.json* ./

#Use npm install (NOT npm ci)
RUN npm install --no-audit --prefer-offline

COPY . .

# Build args
ARG USER_NAME=adish786
ENV PUBLIC_URL=/${USER_NAME}-admin
ENV REACT_APP_API_BASE_URL=/${USER_NAME}-admin/api
RUN npm run build

############################
# Stage 2: Runtime (Nginx)
############################
FROM nginx:1.26-alpine3.20
# Debian Bookworm → patched curl, openssl, expat, libc

# Remove default nginx config
RUN rm /etc/nginx/conf.d/default.conf

# Copy admin nginx config
COPY nginx/default.conf /etc/nginx/conf.d/default.conf

# Copy built React app
COPY --from=builder /app/build /usr/share/nginx/html

# Security hardening (non-root)
RUN adduser --system --no-create-home --group nginxuser && \
    chown -R nginxuser:nginxuser /usr/share/nginx/html

USER nginxuser

EXPOSE 80

# Healthcheck WITHOUT curl/wget
HEALTHCHECK --interval=15s --timeout=5s --retries=3 \
  CMD nginx -t || exit 1

CMD ["nginx", "-g", "daemon off;"]
