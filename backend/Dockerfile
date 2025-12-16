# shopNow backend (Node.js API)

FROM node:20-bookworm-slim

WORKDIR /app

# Install minimal runtime deps
RUN apt-get update && \
    apt-get install -y --no-install-recommends wget ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production --no-audit --prefer-offline

# Copy app source
COPY . .

# Create non-root system user (Debian compatible)
RUN groupadd --system appgroup && \
    useradd --system --gid appgroup --no-create-home \
    --shell /usr/sbin/nologin appuser && \
    chown -R appuser:appgroup /app

USER appuser

ENV NODE_ENV=production
ENV PORT=5000

EXPOSE 5000

HEALTHCHECK --interval=15s --timeout=5s --retries=3 \
  CMD wget -qO- http://127.0.0.1:${PORT}/health || exit 1

CMD ["node", "server.js"]
