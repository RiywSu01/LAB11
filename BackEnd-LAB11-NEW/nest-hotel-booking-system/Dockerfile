# ============================================

# Stage 1: Install ALL dependencies & build

# ============================================

FROM node:20-alpine AS builder

WORKDIR /app

# Copy dependency manifests first (layer caching)

COPY package.json package-lock.json ./

# Install all dependencies (including devDependencies for building)

RUN npm ci

# Copy config files

COPY tsconfig.json tsconfig.build.json nest-cli.json prisma.config.ts ./

#Generate prisma client

COPY prisma ./prisma/

RUN npx prisma generate

#Copy source code

COPY src ./src

# Build the NestJS project

RUN npm run build

# ============================================

# Stage 2: Production-only runtime

# ============================================

FROM node:20-alpine AS production

# Create non-root user

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy dependency manifests

COPY package.json package-lock.json ./

#Add prisma for runtime
COPY prisma ./prisma/
COPY prisma.config.ts ./

# Install production dependencies ONLY

RUN npm ci --omit=dev

#generate Prisma client for production

RUN npx prisma generate

# Copy compiled output from builder stage

COPY --from=builder /app/dist ./dist

# Create uploads directory and assign ownership

# This for adminUploadImage() in roomService, to make non-root-user to able to create the /uploads.

# Let user create by themselves because if copy local uploads folder into the image, every time that build app, it permanently baking all the images in the local test images into the source code of container. This makes Docker image massive and bloated.

RUN mkdir -p ./uploads && chown -R appuser:appgroup /app/uploads && chmod -R 755 /app/uploads

# Switch to non-root user

USER appuser

# Document the port

EXPOSE 3000

# Health check

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \

    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Start the application

CMD ["sh", "-c", "npx prisma db push && node dist/src/main.js"]