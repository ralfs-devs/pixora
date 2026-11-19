# ---------- build stage ----------------
    FROM node:24.22.3-alpine3.23 AS builder
    
    RUN corepack enable

    RUN corepack enable && corepack prepare pnpm@10.33.0 --activate
    
    WORKDIR /app
    
    COPY package.json pnpm-lock.yaml ./
    RUN pnpm install --frozen-lockfile
    
    COPY . .
    RUN pnpm run build
    
# ---------- production stage ----------
    FROM nginx:1.30.6-alpine-slim
    
    COPY nginx.conf /etc/nginx/conf.d/default.conf
    COPY --from=builder /app/dist/pixora-frontend/browser /usr/share/nginx/html
    
    EXPOSE 80
    
    HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
      CMD wget -q --spider http://127.0.0.1/ || exit 1
    
    STOPSIGNAL SIGQUIT