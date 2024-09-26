# Dockerizing a PHP Restaurant Management System: A Step-by-Step Guide

In this post, we'll walk through the process of containerizing a PHP-based Restaurant Management System using Docker. We'll be working with the project available at [https://github.com/SAL6910/Restaurant-Management-System-in-PHP.git](https://github.com/SAL6910/Restaurant-Management-System-in-PHP.git). By the end of this guide, you'll have a fully containerized application that's easy to deploy and manage.

## Prerequisites

Before we begin, make sure you have the following installed on your system:
- Docker
- Docker Compose
- Git

## Step 1: Clone the Repository

First, let's clone the project repository:

```bash
git clone https://github.com/SAL6910/Restaurant-Management-System-in-PHP.git
cd Restaurant-Management-System-in-PHP
```

## Step 2: Create a Dockerfile

Create a file named `Dockerfile` in the root of your project directory with the following content:

```dockerfile
# Use an official PHP runtime as a parent image
FROM php:7.4-apache

# Set the working directory in the container
WORKDIR /var/www/html

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    zip \
    unzip

# Install PHP extensions
RUN docker-php-ext-install pdo pdo_mysql mysqli gd

# Enable Apache mod_rewrite
RUN a2enmod rewrite

# Copy the current directory contents into the container
COPY . /var/www/html/

# Set permissions
RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html

# Expose port 80
EXPOSE 80

# Start Apache in the foreground
CMD ["apache2-foreground"]
```

This Dockerfile sets up an Apache web server with PHP 7.4 and installs necessary extensions for our application.

## Step 3: Create a docker-compose.yml File

Next, create a `docker-compose.yml` file in the root of your project directory:

```yaml
version: '3'

services:
  web:
    build: .
    ports:
      - "80:80"
    volumes:
      - .:/var/www/html
    depends_on:
      - db
    environment:
      - DB_HOST=db
      - DB_USER=root
      - DB_PASSWORD=rootpassword
      - DB_NAME=restaurant_db

  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=rootpassword
      - MYSQL_DATABASE=restaurant_db

volumes:
  db_data:
```

This configuration sets up two services: a web server for our PHP application and a MySQL database.

## Step 4: Update the Database Connection File

Modify the `connection/connection.php` file to use environment variables:

```php
<?php
//main connection file for both admin & front end
$servername = getenv('DB_HOST') ?: 'localhost'; //server
$username = getenv('DB_USER') ?: 'root'; //username
$password = getenv('DB_PASSWORD') ?: ''; //password
$dbname = getenv('DB_NAME') ?: 'online_rest';  //database

// Create connection
$db = mysqli_connect($servername, $username, $password, $dbname); // connecting 

// Check connection
if (!$db) {       //checking connection to DB	
    die("Connection failed: " . mysqli_connect_error());
}
?>
```

This allows our application to connect to the database using the environment variables we set in `docker-compose.yml`.

## Step 5: Build and Run the Docker Containers

Now, let's build and start our Docker containers:

```bash
docker-compose up -d --build
```

This command will build the Docker images and start the containers. The first time you run this, it may take a few minutes to download and set up everything.

## Step 6: Import the Database

Once the containers are running, we need to import our database schema. First, copy the SQL file into the MySQL container:

```bash
docker cp SQL/online_rest.sql restaurant-management-system-in-php_db_1:/tmp/
```

Then, execute the SQL file in the MySQL container:

```bash
docker exec -it restaurant-management-system-in-php_db_1 bash
mysql -uroot -prootpassword restaurant_db < /tmp/online_rest.sql
exit
```

## Step 7: Access Your Application

Your Restaurant Management System should now be up and running! You can access it by opening a web browser and navigating to `http://localhost`.

## Conclusion

Congratulations! You've successfully containerized your PHP Restaurant Management System using Docker. This setup makes it easy to deploy your application consistently across different environments and simplifies the development process.

Remember, this is a basic setup and might need further customization based on your specific requirements. Always ensure to follow best practices for security and performance when deploying to a production environment.

Happy coding!
