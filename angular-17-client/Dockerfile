# Build stage
FROM node:20 AS build

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .
RUN npm run build -- --configuration production

# Run stage
FROM nginx:alpine

# Remove default nginx config
RUN rm /etc/nginx/conf.d/default.conf

# Copy the build output to replace the default nginx contents
COPY --from=build /app/dist/your-angular-app /usr/share/nginx/html

# Create a new nginx config file
RUN echo 'server {                                                           \n\
    listen       80;                                                         \n\
    server_name  localhost;                                                  \n\
                                                                             \n\
    location / {                                                             \n\
        root   /usr/share/nginx/html;                                        \n\
        index  index.html index.htm;                                         \n\
        try_files $uri $uri/ /index.html;                                    \n\
    }                                                                        \n\
                                                                             \n\
    error_page   500 502 503 504  /50x.html;                                 \n\
    location = /50x.html {                                                   \n\
        root   /usr/share/nginx/html;                                        \n\
    }                                                                        \n\
}' > /etc/nginx/conf.d/default.conf

# Install curl for health check
RUN apk add --no-cache curl

# Add a health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

# Expose port 80
EXPOSE 80

# Start Nginx server
CMD ["nginx", "-g", "daemon off;"]