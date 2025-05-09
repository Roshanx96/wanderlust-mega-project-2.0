# Step 1: Build stage
FROM node:16 AS build

# Set working directory
WORKDIR /app

# Install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy the rest of the app's source code
COPY . .

# Build the production version of the app
RUN npm run build

# Step 2: Production stage
FROM nginx:alpine

# Copy the built files from the build stage to nginx's public folder
COPY --from=build /app/dist /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start nginx server
CMD ["nginx", "-g", "daemon off;"]
