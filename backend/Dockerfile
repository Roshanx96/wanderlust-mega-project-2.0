# Step 1: Build stage
FROM node:16 AS build

# Set working directory
WORKDIR /app

# Install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy the rest of the app's source code
COPY . .

# Step 2: Production stage
FROM node:16-slim

# Set working directory
WORKDIR /app

# Copy dependencies and app code from the build stage
COPY --from=build /app /app

# Expose port for backend service (usually 3000 for Node.js)
EXPOSE 3000

# Set environment variables (you can override these at runtime)
ENV NODE_ENV=production

# Run the backend service
CMD ["npm", "start"]
