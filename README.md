<div align="center">

# 🐳 Three-Tier Django Notes Application

### Deploy a Django web application with Docker, Docker Compose, Nginx, and MySQL

![Docker](https://img.shields.io/badge/Docker-Containerization-2496ED?logo=docker&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker%20Compose-Multi--Container-2496ED?logo=docker&logoColor=white)
![Django](https://img.shields.io/badge/Django-Python%20Web%20App-092E20?logo=django&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-Reverse%20Proxy-009639?logo=nginx&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-Database-4479A1?logo=mysql&logoColor=white)

**A hands-on Cloud & DevOps project demonstrating how to containerize and run a three-tier web application using a single Docker Compose command.**

</div>

---

## 📌 Project Overview

This project deploys a **Django Notes App** using a three-tier architecture. Each tier runs in its own Docker container, allowing the web server, application, and database to be managed independently.

The project source code is cloned from GitHub, then built and started with Docker Compose.

### 🎯 Objectives

- Containerize a Django application with a custom Dockerfile.
- Use Docker Compose to define and run multiple services.
- Configure Nginx as the web server and reverse proxy.
- Connect Django to a MySQL database.
- Enable communication between containers over a Docker bridge network.
- Verify the application through a browser and add notes.
- Practice troubleshooting common Docker issues, including port conflicts.

## 🏗️ Architecture

```text
                     User / Web Browser
                             |
                             | HTTP
                             v
                    +------------------+
                    |      NGINX       |
                    | Web / Reverse    |
                    |      Proxy       |
                    |   nginx_cont     |
                    +------------------+
                             |
                             | Proxy request
                             v
                    +------------------+
                    |   DJANGO APP     |
                    | Application Tier |
                    |   django_cont    |
                    |   Port 8000      |
                    +------------------+
                             |
                             | SQL / MySQL
                             v
                    +------------------+
                    |      MYSQL       |
                    |   Database Tier  |
                    |     db_cont      |
                    |   Port 3306      |
                    +------------------+

      All services communicate over a Docker Compose bridge network:
                django-notes-app_notes-app-nw
```

### How the three tiers work

| Tier | Technology | Container | Responsibility |
|---|---|---|---|
| Web tier | Nginx | `nginx_cont` | Receives HTTP requests and acts as a web server/reverse proxy |
| Application tier | Django / Python | `django_cont` | Runs application logic and serves the Notes application on port 8000 |
| Database tier | MySQL | `db_cont` | Stores the application's notes data and listens internally on port 3306 |

## 🧰 Technology Stack

| Technology | Version / Image in the project | Purpose |
|---|---|---|
| Docker Engine | Docker | Runs containers |
| Docker Compose | `docker-compose` | Defines and starts the multi-container application |
| Python | `python:3.9` | Runtime for Django |
| Nginx | `nginx:1.23.3-alpine` | Web server and reverse proxy |
| MySQL | `mysql:latest` | Relational database |
| Git | Git | Clones the application repository |
| Vim | Vim | Edits configuration and project files |

## 🚀 Deploy the Project

### Prerequisites

- An Ubuntu environment such as Killercoda.
- Docker Engine and Docker Compose installed and working.
- Git installed.
- Browser access to the environment's exposed ports.

> **Note:** These steps follow the documented lab setup. Confirm Docker and Compose are installed before starting.

### Step 1: Create a working directory

```bash
mkdir docker-project
cd docker-project
```

### Step 2: Clone the GitHub repository

```bash
git clone https://github.com/thawaresameer715-cyber/django-notes-app.git
cd django-notes-app
ls
```

### Step 3: Review the configuration files

The project uses these main files:

- `Dockerfile` — defines how the Django application image is built.
- `docker-compose.yml` — defines the `db`, `django_app`, and `nginx` services, network, and port mappings.
- `.env` — stores environment variables such as database configuration.
- `nginx/` — contains the Nginx configuration/build files.

Review the existing files and update values only when required by your environment:

```bash
vim Dockerfile
vim docker-compose.yml
vim .env
```

**Security reminder:** Do not commit real passwords, API keys, or other secrets in `.env`. Use a safe example file such as `.env.example` for documented variable names, and keep the real `.env` out of version control.

### Step 4: Check Docker networks

```bash
docker network ls
```

Docker Compose creates a project network for the services. In this lab, the network is named:

```text
django-notes-app_notes-app-nw
```

### Step 5: Build and start all three services

From the directory containing `docker-compose.yml`, run:

```bash
docker-compose up -d
```

This command builds the required images when needed, creates the Compose network, and starts the database, Django, and Nginx containers in detached mode.

To rebuild after changing application or image configuration:

```bash
docker-compose up -d --build
```

### Step 6: Verify the containers

```bash
docker ps -a
```

The documented lab output showed the following services running:

| Container | Expected state | Port information |
|---|---|---|
| `nginx_cont` | Up | Host port `80` → container port `80` |
| `django_cont` | Up (healthy) | Host port `8000` → container port `8000` |
| `db_cont` | Up (healthy) | MySQL port `3306` available internally |

Health status and exact port mappings depend on your Compose configuration and environment.

### Step 7: Open the application

In the documented lab, the Django Notes page was opened through the environment on **port 8000**. If your environment provides a port-forwarding or preview feature, use it to open the exposed port.

The Nginx service is also mapped to host port `80` in the documented configuration. Use the entry point intended by your environment and Nginx configuration.

**Expected result:** The My Notes page loads in the browser. Add a couple of notes and confirm they appear in the application.

## ✅ Verification Commands

```bash
# List running containers
docker ps

# List all containers, including stopped/created ones
docker ps -a

# List local images
docker images

# List Docker networks
docker network ls

# Follow logs from all Compose services
docker-compose logs -f

# Follow logs for an individual service
docker-compose logs -f django_app
docker-compose logs -f nginx
docker-compose logs -f db

# Open a shell inside the Django container
docker exec -it django_cont bash
```

Use `Ctrl+C` to stop following logs; this does not normally stop the containers.

## 🧯 Troubleshooting

### 1. Error: `port is already allocated`

**Example:**

```text
Bind for 0.0.0.0:8000 failed: port is already allocated
```

**Cause:** The Compose stack is already using host port `8000`, and another container is trying to bind to the same port.

**Solutions:**

- Prefer the existing Compose deployment instead of launching a duplicate container.
- Stop and remove the unnecessary container if it is safe to do so.
- Or map the new container to a different free host port, for example:

```bash
docker run -d -p 8001:8000 notes-app:latest
```

Remove an unused container only after confirming it is not needed:

```bash
docker ps -a
docker rm <container_id>
```

### 2. Application does not open

- Check container status with `docker ps -a`.
- Review service logs using `docker-compose logs -f`.
- Confirm the expected host port is published in `docker-compose.yml`.
- In Killercoda or another lab, ensure you are using its correct port-preview mechanism.

### 3. Changes are not reflected

If you changed application code or a Dockerfile, rebuild and restart as appropriate:

```bash
docker-compose up -d --build
```

Check logs and refresh the browser after the services restart.

### 4. Django cannot connect to MySQL

- Confirm `db_cont` is running and healthy.
- Review database service logs.
- Check the database host, database name, username, password, and other environment variables in the Compose configuration.
- Within the Compose network, use the configured database **service name** as the hostname, rather than assuming `localhost` means the MySQL container.

## 🧹 Stop the Project

To stop the application and remove the containers and Compose-created network:

```bash
docker-compose down
```

This command does not normally delete named volumes. Inspect the Compose configuration before cleanup if you need to preserve database data or intentionally remove it.

## 🧠 Key Learnings

- Understood the web, application, and database tiers.
- Built a custom Django image with a Dockerfile.
- Used Docker Compose to manage multiple containers with one command.
- Learned how services communicate over a user-defined Docker network.
- Separated configuration from application code using environment variables.
- Verified application behavior through the browser.
- Diagnosed a host-port conflict and learned safe ways to resolve it.

## 🔮 Possible Improvements

These are potential future enhancements, not features claimed as completed in this lab:

- Add persistent, explicitly configured storage for MySQL.
- Add health checks and service dependency conditions in Compose.
- Use a production-ready Django application server behind Nginx.
- Add a CI/CD pipeline to build and test images automatically.
- Pin image versions instead of relying on `mysql:latest`.
- Deploy the same architecture to a cloud environment.

## 🏁 Conclusion

The Django Notes application was deployed as a three-tier Docker project with Nginx, Django, and MySQL running in separate containers managed by Docker Compose. The documented verification confirmed that the Notes page loaded and notes could be added and displayed.

This project provides practical experience with containerization, multi-container orchestration, Docker networking, environment configuration, and troubleshooting—important foundations for Cloud and DevOps engineering.

# 👨‍💻 Author

**Avishkar Thorave**

- LinkedIn: www.linkedin.com/in/avishkar-thorve-a77a8b2a1
- GitHub: https://github.com/Avishkar5658

---

<div align="center">

**Cloud & DevOps Learning Project** ☁️

*Build • Containerize • Connect • Verify*

</div>
