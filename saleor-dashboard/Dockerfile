#Builder
FROM node:18-alpine AS builder

# Define the working directory
WORKDIR /app

# Install bash using apk
RUN apk add --no-cache bash

# Manage Dependencies
COPY package.json package-lock.json /app/

# Install the dependencies using npm ci
RUN npm ci --legacy-peer-deps 

# Set CI environment to 1 (to avoid prompts during the build)
ENV CI=1

# Copy source files in separate steps
COPY nginx/ /app/nginx/
COPY assets/ /app/assets/
COPY locale/ /app/locale/
COPY src/ /app/src/
COPY scripts/ /app/scripts/

# Copy individual configuration files
COPY vite.config.js /app/
COPY tsconfig.json /app/
COPY schema.graphql /app/
COPY introspection.json /app/
COPY *.d.ts /app/
COPY .featureFlags/ /app/.featureFlags/

# Configure Environment Variables
ARG API_URL
ENV API_URL=$API_URL

ARG APP_MOUNT_URI
ENV APP_MOUNT_URI=$APP_MOUNT_URI

ARG APPS_MARKETPLACE_API_URL
ENV APPS_MARKETPLACE_API_URL=$APPS_MARKETPLACE_API_URL

ARG APPS_TUNNEL_URL_KEYWORDS
ENV APPS_TUNNEL_URL_KEYWORDS=$APPS_TUNNEL_URL_KEYWORDS

ARG STATIC_URL
ENV STATIC_URL=$STATIC_URL

ARG SKIP_SOURCEMAPS
ENV SKIP_SOURCEMAPS=$SKIP_SOURCEMAPS

ARG LOCALE_CODE
ENV LOCALE_CODE=$LOCALE_CODE

# Build the Application
RUN npm run build 

# Production stage
FROM nginx:stable-alpine AS runner

# Set working directory in production image
WORKDIR /app 

# Copy built files from the builder stage
COPY --from=builder /app/build/ /app/

# Copy the Nginx config
COPY ./nginx/default.conf  /etc/nginx/conf.d/default.conf

# Copy and ensure the environment variable script is executable
COPY ./nginx/replace-env-vars.sh /docker-entrypoint.d/50-replace-env-vars.sh
RUN chmod +x /docker-entrypoint.d/50-replace-env-vars.sh

# Documented port
EXPOSE 80

# Make sure nginx runs in the foreground to keep the container alive
CMD ["nginx", "-g", "daemon off;"]
