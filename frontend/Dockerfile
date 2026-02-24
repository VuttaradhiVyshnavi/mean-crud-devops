# Stage 1 - Build Angular App
FROM node:18-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

RUN npm run build --configuration=production

# Stage 2 - Serve with Nginx
FROM nginx:alpine

# Copy built Angular files (output path from angular.json: dist/angular-15-crud)
COPY --from=build /app/dist/angular-15-crud /usr/share/nginx/html

# Copy custom nginx config for Angular routing support
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]