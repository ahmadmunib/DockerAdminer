# Docker Adminer Setup

This project provides a Docker Compose configuration for running Adminer, a lightweight database management tool, with PostgreSQL.

## Project Overview

Adminer is a full-featured database management tool available for MySQL, PostgreSQL, SQLite, MS SQL, Oracle, and more. It is extremely lightweight and easy to use, making it perfect for quick database management tasks.

This setup includes:
- Adminer web interface
- PostgreSQL database
- Custom bridge network for secure communication

## Prerequisites

- Docker Engine
- Docker Compose
- UFW (Uncomplicated Firewall) - for security configuration

## Setup Instructions

1. Clone this repository:
   ```bash
   git clone <repository-url>
   cd Docker/Adminer
   ```

2. Copy the environment variables file:
   ```bash
   cp .env.example .env
   ```

3. Edit the `.env` file with your specific database credentials:
   ```bash
   nano .env
   ```

4. Start the services:
   ```bash
   docker-compose up -d
   ```

5. Access Adminer at `http://localhost:8080`

## UFW Security Configuration

For security reasons, this setup includes instructions for configuring UFW (Uncomplicated Firewall) to restrict access to the PostgreSQL port (5432) to only the Adminer container.

### Current Setup
By default, PostgreSQL port 5432 might be open to all incoming connections, which is a security risk. The recommended configuration restricts access to only the Docker container network.

### Recommended UFW Configuration
After starting the Docker services, you can configure UFW as follows:

1. Check your Docker network details:
   ```bash
   docker network inspect local_network
   ```
   
2. Find the subnet used by your Docker network (typically something like `172.22.0.0/16`)

3. Deny all connections to PostgreSQL port (remove any existing rules):
   ```bash
   sudo ufw deny 5432
   ```

4. Allow connections only from the Docker network subnet:
   ```bash
   sudo ufw allow from 172.22.0.0/16 to any port 5432
   ```

This ensures that only the Adminer container can connect to the PostgreSQL database, improving security by preventing direct access from other sources.

### Finding Your Docker Network Subnet

To determine the correct subnet for your Docker network:

1. After starting the containers, inspect the network:
   ```bash
   docker network inspect local_network
   ```

2. Look for the `Subnet` field in the output, which will be in CIDR notation (e.g., `172.22.0.0/16`)

3. Use that subnet in your UFW allow rule

## Docker Services

This setup includes two services:

### 1. PostgreSQL
- Image: `postgres:15`
- Port: 5432 (internal)
- Environment variables loaded from `.env` file
- Named volume for data persistence

### 2. Adminer
- Image: `adminer:latest`
- Port: 8080 (external access)
- Connected to the same Docker network as PostgreSQL
- No authentication required (connects via internal Docker network)

## Stopping the Services

To stop and remove the containers:
```bash
docker-compose down
```

To stop and remove containers along with volumes:
```bash
docker-compose down -v
```

## Troubleshooting

### Adminer can't connect to PostgreSQL
- Verify that both services are on the same Docker network
- Check that PostgreSQL is running and accepting connections
- Ensure the database name, user, and password are correct in the `.env` file

### Port already in use
- Change the external port mapping in `docker-compose.yml` if 8080 is already in use

### UFW Configuration Issues
- Ensure Docker network subnet is correctly identified
- Make sure the allow rule is more specific than any deny rules
- Check UFW status with `sudo ufw status`

## Security Notes

- Never commit the `.env` file to version control (it's included in `.gitignore` by default)
- Regularly update the Docker images to include security patches
- Consider using Docker secrets for sensitive data in production environments
- The UFW configuration is essential for production environments to prevent unauthorized database access

## File Structure

- `docker-compose.yml`: Defines the services and their configurations
- `.env`: Contains environment variables (not tracked by git)
- `.env.example`: Provides template for environment variables
- `.gitignore`: Specifies files/directories to ignore in git
- `README.md`: This file