# Apache Superset on Docker (Ubuntu EC2)

## Overview
This project documents the deployment of Apache Superset — an open-source business intelligence and data visualisation platform — inside a Docker container hosted on an Ubuntu EC2 instance on AWS.

---

## Architecture

```
┌──────────────────────────────────────────┐
│           AWS EC2 Instance               │
│           (Ubuntu Server)                │
│                                          │
│  ┌────────────────────────────────────┐  │
│  │         Docker Engine              │  │
│  │                                    │  │
│  │  ┌──────────────────────────────┐  │  │
│  │  │   Apache Superset Container  │  │  │
│  │  │   Port: 8088                 │  │  │
│  │  └──────────────────────────────┘  │  │
│  │                                    │  │
│  │  ┌──────────┐  ┌───────────────┐   │  │
│  │  │ PostgreSQ│  │    Redis      │   │  │
│  │  │(metadata)│  │  (caching)    │   │  │
│  │  └──────────┘  └───────────────┘   │  │
│  └────────────────────────────────────┘  │
│                                          │
│  Security Group: Port 8088 open          │
└──────────────────────────────────────────┘
            │
            ▼
     Browser Access
  http://EC2-IP:8088
```

---

## What Was Done

### 1. EC2 Instance Setup
- Launched Ubuntu Server EC2 instance
- Configured Security Group to allow:
  - Port 22 (SSH)
  - Port 8088 (Superset Web UI)
- Assigned Elastic IP

### 2. Docker Installation on Ubuntu
```bash
# Update packages
sudo apt-get update

# Install Docker
sudo apt-get install -y docker.io docker-compose

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group
sudo usermod -aG docker ubuntu
```

### 3. Apache Superset Deployment via Docker Compose

```yaml
# docker-compose.yml
version: "3"
services:
  superset:
    image: apache/superset
    container_name: superset
    ports:
      - "8088:8088"
    environment:
      - SUPERSET_SECRET_KEY=your_secret_key_here
    depends_on:
      - db
      - redis
    volumes:
      - superset_home:/app/superset_home

  db:
    image: postgres:14
    container_name: superset_db
    environment:
      POSTGRES_DB: superset
      POSTGRES_USER: superset
      POSTGRES_PASSWORD: superset_password
    volumes:
      - db_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    container_name: superset_cache

volumes:
  superset_home:
  db_data:
```

### 4. Superset Initialisation
```bash
# Start containers
docker-compose up -d

# Initialise the database
docker exec -it superset superset db upgrade

# Create admin user
docker exec -it superset superset fab create-admin \
  --username admin \
  --firstname Admin \
  --lastname User \
  --email admin@example.com \
  --password yourpassword

# Load example data (optional)
docker exec -it superset superset load_examples

# Initialise Superset
docker exec -it superset superset init
```

### 5. Access
- Navigate to `http://<EC2-Public-IP>:8088`
- Log in with admin credentials
- Connect data sources, create dashboards and charts

---

## Technologies Used
- AWS EC2 (Ubuntu Server)
- Docker & Docker Compose
- Apache Superset
- PostgreSQL (metadata database)
- Redis (caching)
- Linux (Bash)

---

## Key Outcomes
- Fully functional BI platform deployed on AWS with zero licensing cost
- Containerised deployment ensures portability and easy updates
- Enabled organisation to build interactive dashboards from their data
- Isolated services using Docker Compose for maintainability
