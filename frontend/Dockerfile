# ----------------------------
# 1) Build the React frontend
# ----------------------------
FROM node:20-alpine AS build
WORKDIR /app

# Install deps
COPY package*.json ./
# Use `npm ci` if a lockfile exists; otherwise fallback to install.
# (Keeps layers small & reproducible)
RUN if [ -f package-lock.json ]; then npm ci; else npm install; fi

# Copy source and build
COPY . .

# If this project uses CRA, `npm run build` outputs to /build.
# If it uses Vite, output is /dist (see copy path below with both options noted).
RUN npm run build

# ----------------------------
# 2) Serve with Nginx
# ----------------------------
FROM nginx:1.27-alpine

# Copy a minimal SPA config (adds history API fallback to index.html)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# React (CRA) build output:
COPY --from=build /app/build /usr/share/nginx/html
# If this project uses Vite instead, comment the line above and uncomment:
# COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

# Optional basic healthcheck
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://127.0.0.1/ >/dev/null 2>&1 || exit 1

CMD ["nginx", "-g", "daemon off;"]
