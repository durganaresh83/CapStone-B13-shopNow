# frontend/Dockerfile
############################
# Stage 1: Build (React)
############################
FROM node:20-bookworm-slim AS builder
WORKDIR /app

# Install build deps only in builder
RUN apt-get update && \
    apt-get install -y --no-install-recommends python3 make g++ && \
    rm -rf /var/lib/apt/lists/*

# Copy package files
COPY package*.json ./
RUN npm ci --no-audit --prefer-offline

# Copy source
COPY . .

# Build args
ARG USER_NAME=aryan
ENV PUBLIC_URL=/${USER_NAME}
ENV REACT_APP_API_BASE_URL=/${USER_NAME}/api

RUN npm run build


############################
# Stage 2: Runtime (Nginx)
############################
FROM nginx:1.26-bookworm

# Remove default nginx config
RUN rm /etc/nginx/conf.d/default.conf

# Copy nginx config
COPY nginx/default.conf /etc/nginx/conf.d/default.conf

# Copy built static app
COPY --from=builder /app/build /usr/share/nginx/html

# Create non-root user
RUN adduser --system --no-create-home --group nginxuser && \
    chown -R nginxuser:nginxuser /usr/share/nginx/html

USER nginxuser

EXPOSE 80

# Healthcheck WITHOUT curl/wget
HEALTHCHECK --interval=15s --timeout=5s --retries=3 \
  CMD nginx -t || exit 1

CMD ["nginx", "-g", "daemon off;"]
