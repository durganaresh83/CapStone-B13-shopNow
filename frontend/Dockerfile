# frontend/Dockerfile
############################
# Stage 1: Build the app
############################
FROM node:20-bookworm-slim AS builder
LABEL org.opencontainers.image.description="shopNow frontend - build stage"
WORKDIR /app

# Install build deps if needed
RUN apk add --no-cache python3 make g++ || true

# Copy package files (leverage cache)
COPY package*.json ./
RUN npm ci --no-audit --prefer-offline

# Copy source and build
COPY . .

# Build argument for dynamic user path
ARG USER_NAME=adish786
ENV PUBLIC_URL=/${USER_NAME}
ENV REACT_APP_API_BASE_URL=/${USER_NAME}/api
RUN npm run build

############################
# Stage 2: Runtime (nginx)
############################
FROM nginx:1.24-alpine
LABEL org.opencontainers.image.maintainer="you@example.com"

# Copy nginx config from build context
COPY nginx/default.conf /etc/nginx/conf.d/default.conf

# Copy built static files from builder
COPY --from=builder /app/build /usr/share/nginx/html

# Expose web port
EXPOSE 80

# Healthcheck
HEALTHCHECK --interval=15s --timeout=5s --start-period=5s --retries=3 \
  CMD wget -qO- --no-check-certificate http://127.0.0.1/health || exit 1

# Run nginx in foreground
CMD ["nginx", "-g", "daemon off;"]