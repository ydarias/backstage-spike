# Base image
FROM node:20-alpine AS builder

# Set working directory
WORKDIR /app

# Copy the package files
COPY package.json yarn.lock ./

# Install dependencies
RUN yarn install --frozen-lockfile

# Copy app source code
COPY packages/app .

# Build the frontend app
RUN yarn tsc
RUN yarn build

# Serve the app using a lightweight server
FROM nginx:1.21
COPY --from=builder /app/packages/app/dist /usr/share/nginx/html

# Expose the default Nginx port
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
