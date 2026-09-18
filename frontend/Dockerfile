# Stage 1: Build the React application
FROM node:18-alpine AS builder

# Set working directory
WORKDIR /app

# Copy package files first
COPY package*.json ./

# Install exact dependencies
RUN npm ci

# Copy application source code
COPY . .

# Build the production application
RUN npm run build


# Stage 2: Serve the application using Nginx
FROM nginx:alpine

# Copy the production build from the builder stage
COPY --from=builder /app/dist /usr/share/nginx/html

# Replace default Nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Nginx listens on port 80
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
