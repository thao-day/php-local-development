# Don't overcomplicate docker locally for PHP developer

## Overview
This project uses Docker to create a development environment with PHP 8.1, Nginx, MongoDB, and Redis.

## Prerequisites
- Docker
- Docker Compose

## Getting Started
1. Clone the repository.
2. Navigate to the project directory.
3. Run `docker-compose up -d` to start the containers.

## Services
- **user-svc**: This service runs a PHP 8.1 application. It is built using the [Dockerfile.php81.fpm](Dockerfile.php81.fpm) and is exposed on port 9000.
- **nginx**: This service runs an Nginx server. It is built using the [Dockerfile.nginx](Dockerfile.nginx) and is exposed on ports 80 and 443.
- **mongodb**: This service runs a MongoDB database. It uses the official MongoDB image and is exposed on port 27017.
- **redis**: This service runs a Redis database. It uses the official Redis image and is exposed on port 6379.

## Volumes
- **mongodb_data**: This volume is used to persist MongoDB data.
- **redis_data**: This volume is used to persist Redis data.

## Configuration
- Nginx configuration is defined in [modified.nginx.conf](modified.nginx.conf) and [user-svc.nginx.conf](user-svc.nginx.conf).
- PHP configuration is defined in [Dockerfile.php81.fpm](Dockerfile.php81.fpm).

## Creating a New PHP Service

If you want to create a new PHP service, you can use `user-svc` as a template. Here are the steps:

1. Create a new Dockerfile for your service, similar to [Dockerfile.php81.fpm](Dockerfile.php81.fpm). Make sure to adjust the PHP version and extensions as needed. You also using [Dockerfile.php81.fpm](Dockerfile.php81.fpm) if still working on PHP 8.1 with new service.

2. Add a new service definition in the `docker-compose.yml` file. Use the `user-svc` service as a reference. Make sure to change the service name, Dockerfile path, and port mappings as needed. e.g
```
    product-svc:
        build:
            context: .
            dockerfile: "Dockerfile.php81.fpm"
        volumes:
            - ../{YOUR_FOLDER}:/var/www
        ports:
            - 9000:9000
        depends_on:
            - mongodb
            - redis
```
Update nginx
```
    nginx:
        build:
            context: .
            dockerfile: "Dockerfile.nginx"
        volumes:
            - ./user-svc.nginx.conf:/etc/nginx/http.d/user-svc.conf
            - ./product-svc.nginx.conf:/etc/nginx/http.d/product-svc.conf
            - ./logs/nginx:/var/log/nginx
        ports:
            - 8082:8082
            - 8083:8083
            - 443:443
        links:
            - user-svc
            - product-svc
```
3. Create a new Nginx configuration file for your service, similar to [user-svc.nginx.conf](user-svc.nginx.conf). Update the `server_name`, `root`, and `fastcgi_pass` directives as needed. E.g:
```
    server {
    listen 8083;
    server_name product-svc;
    location / {
        root /var/www/public; # Point to the public directory
        try_files $uri /index.php$is_args$args; # Try to serve file directly, fallback to index.php
        index index.php;
        location ~ \.php$ {
            fastcgi_pass product-svc:9000;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_path_info;
        }
    }
}
```

4. Add a volume mapping for the new Nginx configuration file in the `nginx` service definition in the `docker-compose.yml` file.

5. Run `docker-compose up -d` to start the new service.

Remember to update the firewall and networking settings as needed to allow traffic to the new service.

## Contributing
Please read CONTRIBUTING.md for details on our code of conduct, and the process for submitting pull requests to us.

## License
This project is licensed under the MIT License - see the LICENSE.md file for details