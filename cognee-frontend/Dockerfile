# --- build stage ---
FROM node:22-alpine AS builder
WORKDIR /app

# Dependências de sistema necessárias para Next/SWC/sharp em Alpine
RUN apk add --no-cache libc6-compat

# Se sua build cair pro "build from source" do sharp/SWC, descomente a linha abaixo:
# RUN apk add --no-cache python3 make g++ pkgconfig

COPY package*.json ./
RUN npm ci --no-audit --no-fund && npm rebuild lightningcss || true

COPY . .

# injeta a URL da API no build (Next lê em build-time)
ARG NEXT_PUBLIC_BACKEND_API_URL
ENV NEXT_PUBLIC_BACKEND_API_URL=$NEXT_PUBLIC_BACKEND_API_URL
ENV NEXT_TELEMETRY_DISABLED=1

RUN npm run build

# --- runtime stage ---
FROM node:22-alpine AS runner
ENV NODE_ENV=production
WORKDIR /app

# O Next só precisa dos artefatos e estáticos
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/package*.json ./
RUN npm ci --omit=dev --no-audit --no-fund

EXPOSE 3000
CMD ["npm", "run", "start"]
