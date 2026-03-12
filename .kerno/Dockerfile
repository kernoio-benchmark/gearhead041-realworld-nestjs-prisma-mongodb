# Multi-stage Dockerfile for NestJS application with Prisma

# Stage 1: Build stage
FROM node:18-slim AS builder

# Install OpenSSL and other dependencies for Prisma
RUN apt-get update -y && \
    apt-get install -y openssl && \
    rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install all dependencies (including devDependencies for build)
RUN npm ci

# Copy application source code
COPY . .

# Generate Prisma Client
RUN npx prisma generate

# Build the application
RUN npm run build

# Stage 2: Production stage
FROM node:18-slim AS production

# Install OpenSSL for Prisma
RUN apt-get update -y && \
    apt-get install -y openssl && \
    rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Set NODE_ENV to production
ENV NODE_ENV=production

# Copy package files
COPY package*.json ./

# Install only production dependencies
RUN npm ci --omit=dev && npm cache clean --force

# Copy Prisma schema for runtime
COPY --from=builder /app/prisma ./prisma

# Generate Prisma Client in production stage
RUN npx prisma generate

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist

# Expose port (NestJS default is 3000)
EXPOSE 3000

# Run the application
CMD ["node", "dist/main"]
