### Step-by-Step Guide to Deploy PHP Application Using Docker on Azure VM

In this guide, we will walk you through the steps required to deploy a PHP application on an **Azure Virtual Machine** using **Docker**. This setup will run a fullstack application, including a **PHP-Apache server** and a **MySQL database**, using Docker containers.

### Prerequisites

- **Azure Virtual Machine** (VM) with **Ubuntu Server** and at least **8 GB RAM**.
- SSH access to the VM.
- **Docker** and **Docker Compose** installed on the VM (we will walk you through that).
  
---

### Step 1: Connect to Your Azure VM

First, connect to your Azure VM using SSH.

```bash
ssh <your-username>@<your-vm-public-ip>
```

Replace `<your-username>` with your Azure VM's username, and `<your-vm-public-ip>` with the public IP address of your VM.

---

### Step 2: Update Your VM

Run the following commands to update all the packages and ensure your VM is up to date.

```bash
sudo apt update -y
```

---

### Step 3: Install Docker

Next, install **Docker** on your Azure VM. Run the following command to install Docker.

```bash
sudo apt install -y docker.io
```

Give your user permission to access Docker without using `sudo`:

```bash
sudo chown $USER /var/run/docker.sock
```

Now verify that Docker is installed correctly:

```bash
docker --version
```

---

### Step 4: Install Docker Compose

Docker Compose is used to manage multi-container Docker applications. You can install Docker Compose using the following commands:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Make the binary executable:

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

Verify the installation:

```bash
docker-compose --version
```

---

### Step 5: Clone the PHP Application Repository

You will need to clone the **Restaurant Management System** from GitHub:

```bash
git clone https://github.com/SAL6910/Restaurant-Management-System-in-PHP.git
```

Once cloned, move into the project directory:

```bash
cd Restaurant-Management-System-in-PHP
```

---

### Step 6: Configure the Dockerfile

In this step, we will create a **Dockerfile** to configure the PHP-Apache server.

Create or edit the `Dockerfile` in the root project directory:

```bash
nano Dockerfile
```

Add the following configuration to the **Dockerfile**:

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

---

### Step 7: Create `docker-compose.yml`

Next, create the `docker-compose.yml` file that defines the services:

```bash
nano docker-compose.yml
```

Add the following content:

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

This file defines two services:

- **web**: Your PHP application running on Apache.
- **db**: A MySQL database service.

---

### Step 8: Configure the Database Connection

Now, configure the database connection in the `connect.php` file:

```bash
cd connection/
rm connect.php
nano connect.php
```

Modify the file to use environment variables for the database connection:

```php
<?php
// Main connection file for both admin & frontend
$servername = getenv('DB_HOST') ?: 'localhost';
$username = getenv('DB_USER') ?: 'root';
$password = getenv('DB_PASSWORD') ?: '';
$dbname = getenv('DB_NAME') ?: 'online_rest';

// Create connection
$db = mysqli_connect($servername, $username, $password, $dbname);

// Check connection
if (!$db) {
    die("Connection failed: " . mysqli_connect_error());
}
?>
```

---

### Step 9: Build and Start the Containers

Now, you can build and run the Docker containers. First, build the Docker image:

```bash
docker-compose up -d --build
```

Check Running Containers: Verify that your containers are running by listing them.
```bash
docker ps
```

This command will spin up the **PHP-Apache** container and the **MySQL** container. The application will be running on port 80 of your Azure VM.

---

### Step 10: Import the Database

To populate the MySQL database, follow these steps:

1. Copy the SQL file from your local machine into the MySQL container:

    ```bash
    docker cp SQL/online_rest.sql restaurant-management-system-in-php-db-1:/tmp/
    ```

2. Access the MySQL container:

    ```bash
    docker exec -it restaurant-management-system-in-php-db-1 bash
    ```

3. Run the following command inside the container to import the SQL data:

    ```bash
    mysql -uroot -prootpassword restaurant_db < /tmp/online_rest.sql
    ```

4. Optionally, verify that the data was imported:

    ```bash
    mysql -uroot -prootpassword
    use restaurant_db;
    SELECT * FROM users LIMIT 5;
    ```

---

### Step 11: Access the Application

To access the application, open your browser and go to:

```
http://<your-vm-public-ip>
```

Replace `<your-vm-public-ip>` with the public IP address of your Azure VM.

You should now be able to see the **Restaurant Management System** up and running.

---

### Step 12: Stopping and Cleaning Up

To stop the running containers, use the following command:

```bash
docker-compose down
```

This will stop and remove the containers but keep the data in the Docker volumes.

---

### Conclusion

You have successfully deployed a **fullstack PHP application** on an **Azure VM** using **Docker**. This guide covered everything from installing Docker and Docker Compose on the Ubuntu server to deploying a PHP-Apache web server with a MySQL database. You can now expand on this by adding more features or services as needed.

If you encounter any issues during setup, double-check the configuration files and ensure your Azure VM's firewall allows HTTP traffic on port 80.
