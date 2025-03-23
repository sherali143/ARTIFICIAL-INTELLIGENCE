# Laravel-React Docker Setup Guide

## Step 1: Create a Dockerfile
Create a `Dockerfile` in your project directory with the following content:

```dockerfile
# Use PHP 8.2-FPM as the base image
FROM php:8.2-fpm

# Set working directory inside the container
WORKDIR /var/www/html

# Install required packages
RUN apt-get update && apt-get install -y \
    curl \
    git \
    zip \
    unzip \
    && curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs

# Install Composer (version 2.5.5)
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=2.5.5

# Install Laravel
RUN composer create-project --prefer-dist laravel/laravel .

# Install Laravel Breeze with React + Inertia
RUN composer require laravel/breeze && \
    php artisan breeze:install react && \
    php artisan migrate --force

# Install Front-End Packages
RUN npm install \
    @headlessui/react \
    react \
    react-dom \
    @vitejs/plugin-react \
    @mantine/core \
    @mantine/hooks \
    @mantine/form \
    @mantine/dates \
    @mantine/charts \
    @mantine/notifications \
    @mantine/code-highlight \
    @mantine/tiptap \
    @mantine/dropzone \
    @mantine/carousel \
    @mantine/spotlight \
    @mantine/modals \
    @mantine/nprogress \
    @mantinex/mantine-logo \
    postcss \
    postcss-preset-mantine \
    postcss-simple-vars

# Expose necessary ports
EXPOSE 8181 5173

# Start Laravel & Vite servers
CMD ["sh", "-c", "php artisan serve --host=0.0.0.0 --port=8181 & npm run dev -- --host"]
```

## Step 2: Build and Run the Container
Execute the following commands in your project directory:

```sh
docker build -t practise-docker-image .
docker run -d -p 8181:8181 -p 5173:5173 --name practise-docker-container practise-docker-image
```

## Step 3: Update `.env` File
Inside the container, update the `.env` file:

```sh
docker exec -it practise-docker-container bash
nano .env
```
Modify the following lines:

```env
APP_URL=http://localhost:8181
```

Clear configuration cache:

```sh
php artisan config:clear
```

## Step 4: Update `vite.config.js`
Modify `vite.config.js` to ensure proper Vite server configuration:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    server: {
        host: '0.0.0.0',
        port: 5173,
        strictPort: true,
        hmr: {
            host: 'localhost',
        },
    },
    plugins: [
        laravel({
            input: 'resources/js/app.jsx',
            refresh: true,
        }),
        react(),
    ],
});
```

## Step 5: Restart the Container
After making the changes, restart the container:

```sh
docker restart practise-docker-container
```

Check logs to verify that everything is running properly:

```sh
docker logs practise-docker-container
```

Now, you should be able to access your Laravel application at `http://localhost:8181` and your Vite-powered frontend at `http://localhost:5173`. 🚀

