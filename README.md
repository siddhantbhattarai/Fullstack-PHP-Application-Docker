### How to Deploy a PHP Full Stack Application on an Azure VM using Docker

In this blog post, we will walk through the step-by-step process of deploying a PHP-based full-stack application, specifically a Restaurant Management System, on an Azure Virtual Machine (VM) using Docker. We'll cover everything from installing Docker to setting up the database and PHP environment.

#### Step 1: Install Docker on Azure VM

Start by logging into your Azure VM, either through SSH or directly from the Azure portal's Cloud Shell.

1. **Update System Packages:**

   Begin by updating the system's package list to ensure that you're installing the latest version of Docker.

   ```bash
   sudo apt update
   ```

2. **Install Docker:**

   Install Docker using the following command:

   ```bash
   sudo apt install -y docker.io
   ```

3. **Change Docker Owner Permissions:**

   In order to avoid permission issues with Docker commands, change the ownership of the Docker socket file.

   ```bash
   sudo chown $USER /var/run/docker.sock
   ```

4. **Verify Docker Installation:**

   Confirm that Docker is installed successfully by checking its version.

   ```bash
   docker --version
   ```

#### Step 2: Clone the Project Repository

Next, clone the GitHub repository for the Restaurant Management System in PHP to your VM.

```bash
git clone https://github.com/SAL6910/Restaurant-Management-System-in-PHP.git
cd Restaurant-Management-System-in-PHP
```

#### Step 3: Install Docker Compose

Docker Compose is a tool that allows you to define and manage multi-container Docker applications. Let’s install it to manage our PHP application and MySQL database.

1. **Download Docker Compose:**

   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```

2. **Set Executable Permissions:**

   Make Docker Compose executable.

   ```bash
   sudo chmod +x /usr/local/bin/docker-compose
   ```

3. **Verify Docker Compose Installation:**

   Confirm the installation by checking the version.

   ```bash
   docker-compose --version
   ```

#### Step 4: Build and Configure Docker Containers

Now that Docker and Docker Compose are installed, it's time to set up the application and database containers.

1. **Dockerfile:**

   This is a simple Dockerfile used to create a custom Docker image for the PHP application.

   ```Dockerfile
   # Use an official PHP runtime as a parent image
   FROM php:7.4-apache

   # Set the working directory in the container
   WORKDIR /var/www/html

   # Install system dependencies and PHP extensions
   RUN apt-get update && apt-get install -y \
       libpng-dev \
       libjpeg-dev \
       libfreetype6-dev \
       zip \
       unzip

   RUN docker-php-ext-install pdo pdo_mysql mysqli gd

   # Enable Apache mod_rewrite
   RUN a2enmod rewrite

   # Copy the application source code
   COPY . /var/www/html/

   # Set permissions
   RUN chown -R www-data:www-data /var/www/html \
       && chmod -R 755 /var/www/html

   # Expose port 80
   EXPOSE 80

   # Start Apache server
   CMD ["apache2-foreground"]
   ```

2. **docker-compose.yml:**

   Define the application and database services in the `docker-compose.yml` file:

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
3. **Update the Database Connection File**

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

4. **Start the Docker Containers:**

   Build and start the application and database containers in detached mode.

   ```bash
   docker-compose up -d --build
   ```

5. **Check Running Containers:**

   Verify that your containers are running by listing them.

   ```bash
   docker ps
   ```

#### Step 5: Import the Database

Now, let's import the SQL database dump into the MySQL container.

1. **Copy the SQL file to the MySQL container:**

   Use the `docker cp` command to copy the SQL file into the MySQL container.

   ```bash
   docker cp SQL/online_rest.sql restaurant-management-system-in-php-db-1:/tmp/
   ```

2. **Import the SQL file:**

   Access the MySQL container and import the database using the MySQL command.

   ```bash
   docker exec -i restaurant-management-system-in-php-db-1 mysql -uroot -prootpassword restaurant_db < /tmp/online_rest.sql
   ```

   *Note: You may receive a warning about using a password on the command line, but this can be ignored for local development purposes.*

#### Step 6: Verify the Deployment

After completing these steps, your PHP application should be up and running, connected to the MySQL database. You can access the application by navigating to your Azure VM’s IP address in a web browser.

To check the imported database, you can access the MySQL container and run a query:

```bash
docker exec -it restaurant-management-system-in-php-db-1 mysql -uroot -prootpassword
mysql> SELECT * FROM users LIMIT 5;
```

#### Conclusion

By following these steps, you have successfully deployed a PHP-based Restaurant Management System on an Azure VM using Docker. This setup ensures easy scalability and modularity, making it an excellent choice for modern web development projects. Keep experimenting with Docker to explore more advanced features like networking, multi-host setups, and Docker Swarm!
