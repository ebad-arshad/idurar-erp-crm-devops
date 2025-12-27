# Stage 1: The Build & Dependency Environment
FROM node:20-alpine AS builder

# Set working directory
WORKDIR /app

# Copy only package files first to leverage Docker cache
COPY package*.json ./

# Install ALL dependencies (including devDependencies for any build steps)
RUN npm ci

# Copy the rest of the source code
COPY . .

# Stage 2: The Production Runner
FROM node:20-alpine

# Set to production to optimize performance and security
# ENV NODE_ENV=production

WORKDIR /app

# Copy only the production-ready node_modules from builder
# Note: In a real pro setup, you'd run 'npm ci --omit=dev' in builder 
# and copy those modules here.
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/src ./src
COPY --from=builder /app/package.json ./package.json

EXPOSE 3000

CMD ["node", "src/server.js"]