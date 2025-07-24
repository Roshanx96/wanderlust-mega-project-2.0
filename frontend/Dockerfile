# Stage 1: Build the frontend
FROM node:18-alpine AS builder
WORKDIR /app

# Copy and install dependencies
COPY package*.json ./
RUN npm ci

# Copy source code and environment
COPY . .
# Inject environment variables into Vite build
ENV NODE_ENV=production

# Vite will read from .env.docker automatically if present
RUN cp .env.docker .env && npm run build

# Stage 2: Serve using NGINX
FROM nginx:stable-alpine AS runner
COPY --from=builder /app/dist /usr/share/nginx/html

# Remove default NGINX config (optional)
RUN rm /etc/nginx/conf.d/default.conf

# Add custom NGINX config (recommended)
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
