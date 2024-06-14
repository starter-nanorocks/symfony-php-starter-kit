# Symfony PHP Starter Kit

This is a starter kit for Symfony 7.1 with the following technologies:

- Docker
- Docker Compose
- PHP 8.2
- Nginx
- Adminer
- PostgreSQL
- MailHog
- Xdebug

## Requirements

- Docker
- Docker Compose

## Getting Started

1. Clone the repository:

   ```sh
   git clone https://github.com/yourusername/symfony-starter-kit.git
   cd symfony-starter-kit
   ```

2. Create environment variables:

   ```sh
   cp .env.dist .env
   ```

3. Build and start the containers:

   ```sh
   docker-compose up --build -d
   ```

4. Install Symfony dependencies:

   ```sh
   docker-compose exec php composer install
   ```

## Services

- **PHP 8.2**: The PHP version used for the application.
- **Nginx**: Serves the Symfony application.
- **PostgreSQL**: The database server.
- **Adminer**: Database management tool accessible at [http://localhost:8080](http://localhost:8080).
- **MailHog**: Email testing tool accessible at [http://localhost:8025](http://localhost:8025).

## Docker Compose Configuration

Here is the `docker-compose.yml` configuration:

```yaml
version: '4.2'

services:
  php-fpm:
    container_name: php
    build:
      context: .
      dockerfile: docker/php/Dockerfile
    volumes:
      - ./symfony:/var/www/html/symfony
      - ./docker/php/conf.d/xdebug.ini:/usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
      - ./docker/php/conf.d/error_reporting.ini:/usr/local/etc/php/conf.d/error_reporting.ini
    networks:
      - elrnero

  nginx:
    container_name: nginx
    image: nginx:stable
    ports:
      - '80:80'
    volumes:
      - ./symfony:/var/www/html/symfony
      - ./docker/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf
    networks:
      - elrnero

  db:
    container_name: postgres
    image: postgres
    restart: always
    shm_size: 128mb
    environment:
      POSTGRES_PASSWORD: secret
    ports:
      - '5432:5432'
    volumes:
      - ./docker/db/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - elrnero
    command: postgres -c 'max_connections=200'

  adminer:
    container_name: adminer
    image: adminer
    restart: always
    environment:
      ADMINER_DEFAULT_DB_HOST: database
      ADMINER_DESIGN: haeckel
      ADMINER_PLUGINS: tables-filter tinymce
    ports:
      - '54320:8080'
    networks:
      - elrnero

  mailhog:
    container_name: mailhog
    image: mailhog/mailhog:latest
    restart: always
    ports:
      - 1025:1025
      - 8025:8025
    networks:
      - elrnero

networks:
  elrnero:
    driver: bridge
```

## Pre-commit Setup with GrumPHP

### Install GrumPHP

Add GrumPHP to your Symfony project using Composer:

```sh
docker-compose exec php composer require --dev phpro/grumphp
```

### Configure GrumPHP

Create a `grumphp.yml` file in the root of your project:

```yaml
grumphp:
    tasks:
        phpcs:
            standard:
                - 'PSR12'
        phpstan:
            autoload_file: ~
            configuration: ~
            level: null
            force_patterns: []
            ignore_patterns: []
            triggered_by: ['php']
            memory_limit: "-1"
            use_grumphp_paths: true
```

### Update composer.json

Add the GrumPHP configuration to your `composer.json` scripts section to automatically run GrumPHP on `git commit`:

```json
"scripts": {
    "post-install-cmd": [
        "vendor/bin/grumphp git:init"
    ],
    "post-update-cmd": [
        "vendor/bin/grumphp git:init"
    ]
}
```

### Initialize GrumPHP

Run the following command to initialize GrumPHP:

```sh
php vendor/bin/grumphp git:init
```

## Database Configuration

Update the `.env` file with your database credentials:

```
DATABASE_URL=postgresql://symfony:symfony@postgres:5432/symfony
```

## Accessing Services

- Symfony application: [http://localhost:8000](http://localhost:8000)
- Adminer: [http://localhost:8080](http://localhost:8080)
- MailHog: [http://localhost:8025](http://localhost:8025)

## License

This project is licensed under the MIT License.
```

This setup ensures that GrumPHP will run PSR-12 code style checks and PHPStan analysis before each commit. Adjust any URLs, repository names, or other specifics to fit your project's setup.
