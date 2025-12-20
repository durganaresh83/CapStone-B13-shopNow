############################
# Stage 1: Build (React)
############################
FROM node:20-bookworm-slim AS builder
WORKDIR /app

# Build dependencies (only in builder)
RUN apt-get update && \
    apt-get install -y --no-install-recommends python3 make g++ && \
    rm -rf /var/lib/apt/lists/*

# Install deps
COPY package.json package-lock.json* ./
RUN npm install --no-audit --prefer-offline

# Source
COPY . .

# Build args
ARG USER_NAME=adish786
ENV PUBLIC_URL=/${USER_NAME}-admin
ENV REACT_APP_API_BASE_URL=/${USER_NAME}-admin/api

RUN npm run build


############################
# Stage 2: Runtime (NGINX)
############################
FROM nginx:1.26-alpine3.20

# 🔥 Disable default entrypoint scripts (they assume root)
RUN rm -rf /docker-entrypoint.d/*

# Copy nginx configs
COPY nginx/nginx.conf /etc/nginx/nginx.conf
COPY nginx/default.conf /etc/nginx/conf.d/default.conf

# Copy built app
COPY --from=builder /app/build /usr/share/nginx/html

# Create non-root user
RUN addgroup -S nginxuser \
    && adduser -S -H -G nginxuser nginxuser \
    && chown -R nginxuser:nginxuser \
       /usr/share/nginx/html \
       /var/cache/nginx \
       /var/log/nginx \
       /tmp

# Run as non-root
USER nginxuser

EXPOSE 80

# Healthcheck
HEALTHCHECK --interval=15s --timeout=5s --start-period=5s --retries=3 \
  CMD wget -qO- --no-check-certificate http://127.0.0.1/health || exit 1

CMD ["nginx", "-g", "daemon off;"]
