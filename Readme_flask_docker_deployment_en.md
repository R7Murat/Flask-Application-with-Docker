## 📚 **Project Assignment: Deploying a Flask Application with Docker, Gunicorn, and Nginx**

### **Project Description:**

In this project, you are expected to deploy a car rental application built with Python Flask onto an EC2 instance using Docker, Gunicorn, Nginx, and Terraform. Your application should include the following components:

1. **Flask Application:** A simple car rental platform built with Python Flask. This platform allows users to rent and view cars.
2. **Docker Compose:** The application should be structured into multiple services, each running independently. These services include:
   - **app**: Flask application with Gunicorn
   - **mysql**: MySQL database
   - **nginx**: Nginx server acting as a reverse proxy for the application.
3. **Nginx Configuration:** Nginx must be configured to route incoming requests from the internet to the Flask application. It can also be set up to support SSL certificates.

### **Project Steps:**

1. **Writing the Dockerfile:**

   - A `Dockerfile` should be written to create a Docker image for the Flask application.
   - Dependencies will be installed and the app will be launched with Gunicorn.
   - Key components of the `Dockerfile` include:
     - Use of Python 3.9 base image
     - Installation of necessary dependencies
     - Launching the Flask app using Gunicorn

   **Dockerfile Example:**

   ```dockerfile
   # Base image
   FROM python:3.9

   # Set the working directory in the container
   WORKDIR /app

   # Copy the current directory contents into the container
   COPY . /app

   # Install dependencies
   RUN pip install --no-cache-dir -r requirements.txt

   # Expose port 8000 for Gunicorn
   EXPOSE 8000

   # Command to run the application using Gunicorn
   CMD ["gunicorn", "-b", "0.0.0.0:8000", "wsgi:app"]
   ```

2. **Docker Compose File:**

   - Docker Compose will manage the `mysql`, `app` (Flask + Gunicorn), and `nginx` (Reverse Proxy) services.
   - Each service will be configured independently.

   **Docker Compose Example:**

   ```yaml
   version: '3'
   services:
     app:
       build: .
       container_name: flask_app
       environment:
         - DB_HOST=mysql
         - DB_USER=root
         - DB_PASSWORD=rootpassword
         - DB_NAME=arac_kiralama
       depends_on:
         - mysql
       networks:
         - app-network
       ports:
         - "8000:8000"

     mysql:
       image: mysql:5.7
       container_name: mysql_container
       environment:
         MYSQL_ROOT_PASSWORD: rootpassword
         MYSQL_DATABASE: arac_kiralama
       networks:
         - app-network
       ports:
         - "3306:3306"

     nginx:
       image: nginx:latest
       container_name: nginx_reverse_proxy
       volumes:
         - ./nginx.conf:/etc/nginx/nginx.conf
       ports:
         - "80:80"
       depends_on:
         - app
       networks:
         - app-network

   networks:
     app-network:
       driver: bridge
   ```

3. **Nginx Configuration:**

   - Nginx will receive requests and route them to the Flask app running with Gunicorn.

   **nginx.conf Example:**

   ```nginx
   server {
       listen 80;
       server_name ${DOMAIN_NAME} www.${DOMAIN_NAME};

       location / {
           proxy_pass http://app:8000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

4. **Deploying to EC2 with Terraform:**

   - Terraform will launch an EC2 instance on AWS and run Docker Compose on it.

   **Terraform Example:**

   ```hcl
   provider "aws" {
     region = "us-east-1"
   }

   resource "aws_instance" "flask_app" {
     ami           = "ami-0c55b159cbfafe1f0" # Replace with a suitable AMI ID
     instance_type = "t2.micro"

     tags = {
       Name = "FlaskAppInstance"
     }

     user_data = <<-EOF
       #!/bin/bash
       apt update -y
       apt install -y docker.io
       apt install -y docker-compose
       cd /home/ubuntu
       git clone https://github.com/username/car-rental.git
       cd car-rental
       docker-compose up -d
     EOF
   }
   ```

### **Assignment Requirements:**

1. **Creating Dockerfile and Docker Compose Files:**

   - Write a `Dockerfile` to run the Flask application in Docker.
   - Use `docker-compose.yml` to configure MySQL, Nginx, and Flask services.

2. **Running the Application:**

   - Start the application using Docker Compose.
   - Ensure that Nginx works correctly with the Flask app.

3. **Deploying to EC2 with Terraform:**

   - Launch an EC2 instance using Terraform and run Docker Compose on it.

4. **Uploading the Project to a Repository:**

   - Upload all project files to a GitHub repository.

