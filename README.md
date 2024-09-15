Here’s a simplified and detailed documentation for your project that explains each part of the Docker Compose setup, the services, and how everything ties together.

---

# Project Documentation

## Overview

This project consists of three main services:

1. **Proxy (Nginx)**: Acts as a reverse proxy and handles HTTPS connections, forwarding traffic to the backend service.
2. **Backend**: A Go-based application that serves business logic.
3. **Database (MySQL)**: Stores data for the backend service.

The services are managed with Docker Compose, which allows you to build, run, and manage them easily in a single configuration.

---

## File Structure

Your project is organized into directories and files as follows:

```
.
├── backend/
│   ├── Dockerfile         # Dockerfile for building the Go backend service
│   ├── main.go            # Go code for the backend service
│   ├── go.mod             # Go dependencies
│   ├── go.sum             # Go dependencies checksum
├── proxy/
│   ├── Dockerfile         # Dockerfile for building the Nginx proxy
│   ├── nginx.conf         # Nginx configuration for reverse proxy and SSL
│   ├── generate-ssl.sh    # Script to generate SSL certificates
├── db-password.txt        # Secret file containing the MySQL root password
├── docker-compose.yml     # Docker Compose configuration for all services
```

---

## Services Description

### 1. **Proxy (Nginx)**

The `proxy` service is an Nginx reverse proxy configured to handle HTTPS traffic on port 443. It forwards requests to the backend service running on port 8000. It also includes an SSL certificate generation script that creates self-signed certificates for secure HTTPS communication.

**Key Points:**
- The SSL certificates are stored in `/etc/nginx/ssl/`.
- The Nginx configuration file (`nginx.conf`) defines the reverse proxy behavior, including how to forward traffic to the backend and manage SSL.

**Files:**
- `proxy/Dockerfile`: Builds the Nginx image and ensures SSL certificates are generated before starting Nginx.
- `proxy/nginx.conf`: Configures Nginx to listen on port 443 (HTTPS) and forward traffic to `backend:8000`.
- `proxy/generate-ssl.sh`: Script to generate SSL certificates.

### 2. **Backend (Go Service)**

The `backend` service is a Go application that serves the core business logic. It listens on port 8000 and responds to requests forwarded by the `proxy` service.

**Key Points:**
- The Go application is built using a multi-stage Docker build process to create a minimal container image.
- The backend exposes port 8000 for communication with the proxy.

**Files:**
- `backend/Dockerfile`: Defines how to build the Go application.
- `backend/main.go`: The source code for the backend application.
- `backend/go.mod` and `go.sum`: Manage Go dependencies.

### 3. **Database (MySQL)**

The `db` service is a MySQL database used to store the backend’s data. The MySQL root password is securely managed using Docker secrets.

**Key Points:**
- The database service uses Docker volumes to persist data across restarts.
- The MySQL root password is stored in the `db-password.txt` file, and Docker Compose loads this password securely using Docker secrets.

**Files:**
- `db-password.txt`: A file containing the root password for MySQL.
- The MySQL service uses the `mysql:8.0` image from Docker Hub.

---

## Docker Compose Configuration

The `docker-compose.yml` file is the central configuration for running all services together.

### Key Sections:

1. **Version**: The Compose file uses version 3.7 of Docker Compose.

2. **Services**: 
   - **proxy**: 
     - Builds the Nginx proxy from the `proxy/` directory.
     - Exposes port 443 for HTTPS.
     - Mounts the SSL certificate directory (`/etc/nginx/ssl/`).
     - Depends on the backend service to ensure it starts after the backend is ready.
   - **backend**:
     - Builds the Go application from the `backend/` directory.
     - Exposes port 8000 for internal communication with the proxy.
     - Depends on the `db` service.
     - Uses a Docker secret to securely access the MySQL password.
   - **db**:
     - Uses the official `mysql:8.0` image.
     - Mounts a volume to persist data across container restarts.
     - Uses Docker secrets to securely load the MySQL root password.

3. **Secrets**:
   - `db-password`: Loads the MySQL root password from the `db-password.txt` file and makes it available to both the `db` and `backend` services.

4. **Volumes**:
   - `db_data`: A Docker volume that stores MySQL data, ensuring it persists even if the container is restarted.

### Sample `docker-compose.yml`:

```yaml
version: '3.7'

services:
  proxy:
    build: proxy/
    image: nginx_proxy
    ports:
      - "443:443"
    volumes:
      - /etc/nginx/ssl/:/etc/nginx/ssl/
    depends_on:
      - backend

  backend:
    build:
      context: backend
      dockerfile: Dockerfile
    image: backend_service
    expose:
      - "8000"
    depends_on:
      - db
    secrets:
      - db-password

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db-password
      MYSQL_DATABASE: example
    volumes:
      - db_data:/var/lib/mysql
    secrets:
      - db-password

secrets:
  db-password:
    file: ./db-password.txt

volumes:
  db_data:
```

---

## How to Run the Project

1. **Clone the Project**:
   ```bash
   git clone https://your-repository-url
   cd your-project-directory
   ```

2. **Create the MySQL Password File**:
   - Create a file named `db-password.txt` in the project root and store the MySQL root password there:
     ```
     echo "your-mysql-root-password" > db-password.txt
     ```

3. **Build and Start the Services**:
   - Use Docker Compose to build and start all services:
     ```bash
     docker-compose up --build
     ```

4. **Access the Application**:
   - Your Nginx proxy will be available on `https://localhost:443`. The Nginx proxy handles SSL termination, and requests will be forwarded to the backend service.

5. **Stop the Services**:
   - To stop and remove the containers, run:
     ```bash
     docker-compose down
     ```

---
