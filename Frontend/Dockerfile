# ============================================================
# Stage 1: Build
# Uses BUILDPLATFORM to run the build natively (ARM or X86),
# and targets the final image for both ARM64 and AMD64.
# ============================================================
FROM --platform=$BUILDPLATFORM node:20-alpine AS builder

WORKDIR /app

# Install dependencies first (layer caching)
COPY package*.json ./
RUN npm ci --prefer-offline || npm install

# Copy source and build
COPY . .
RUN npm run build

# ============================================================
# Stage 2: Production (multi-arch nginx)
# nginx:alpine supports both linux/amd64 and linux/arm64
# ============================================================
FROM --platform=$TARGETPLATFORM nginx:alpine

# Remove default nginx config
RUN rm /etc/nginx/conf.d/default.conf

# Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy built assets from builder stage
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
