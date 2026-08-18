# 🐳 Full Stack Application with Docker Compose

A containerized full-stack application demonstrating how a **frontend, backend, and PostgreSQL database** can be developed and run as independent Docker services using Docker Compose.

The project focuses on practical Docker and containerization concepts including **Dockerfiles, multi-container architecture, service networking, persistent database storage, environment configuration, port mapping, and Docker Compose orchestration**.

---

## 🚀 Project Overview

This project consists of three services:

* **Frontend** — React/Vite application served through Nginx
* **Backend** — Node.js application
* **Database** — PostgreSQL 16 running in an Alpine-based container

All services are managed through a single `docker-compose.yml` configuration.

```text
                         Docker Compose
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
        ┌──────────┐    ┌──────────┐    ┌─────────────┐
        │ Frontend │───►│ Backend  │───►│ PostgreSQL  │
        │ React    │    │ Node.js  │    │     16      │
        └──────────┘    └──────────┘    └─────────────┘
             │                │                │
           :3000            :5001           :5432
                              │
                              ▼
                       Docker Network
                              │
                              ▼
                       Persistent Volume
                            pgdata
```

---

## ✨ Features

* **Multi-container architecture** with separate frontend, backend, and database services
* **Dockerized frontend** using a multi-stage Docker build
* **Dockerized backend** using a Node.js Alpine image
* **PostgreSQL 16** running as an independent database service
* **Docker Compose orchestration** for the complete stack
* **Custom Docker bridge network** for internal service communication
* **Persistent PostgreSQL storage** using a named Docker volume
* **Environment-based database configuration**
* **Service dependency management** using `depends_on`
* **Host-to-container port mapping**
* **Nginx** used to serve the production frontend build
* Services can communicate using Docker service names instead of hardcoded IP addresses

---

## 🏗️ Architecture

The application follows a simple three-service architecture:

```text
┌─────────────────────────────────────────────────┐
│                 Docker Compose                  │
│                                                 │
│   ┌─────────────┐                               │
│   │  Frontend   │                               │
│   │ React/Vite  │                               │
│   │    Nginx    │                               │
│   │    :80      │                               │
│   └──────┬──────┘                               │
│          │                                      │
│          │ HTTP                                 │
│          ▼                                      │
│   ┌─────────────┐                               │
│   │   Backend   │                               │
│   │   Node.js   │                               │
│   │    :5000    │                               │
│   └──────┬──────┘                               │
│          │                                      │
│          │ PostgreSQL                           │
│          ▼                                      │
│   ┌─────────────┐                               │
│   │ PostgreSQL  │                               │
│   │     16      │                               │
│   │    :5432    │                               │
│   └──────┬──────┘                               │
│          │                                      │
│          ▼                                      │
│   ┌─────────────┐                               │
│   │   pgdata    │                               │
│   │   Volume    │                               │
│   └─────────────┘                               │
│                                                 │
│          my-custom-network                      │
└─────────────────────────────────────────────────┘
```

---

## 🔗 Service Communication

All three services are connected to a custom Docker network:

```yaml
networks:
  my-custom-network:
    driver: bridge
```

Because the services share the same network, the backend can connect to PostgreSQL using:

```text
database:5432
```

instead of using:

```text
localhost:5432
```

The database hostname is configured through:

```text
DB_HOST=database
```

Docker's internal DNS resolves the service name `database` to the PostgreSQL container.

---

## 🐳 Docker Services

### Frontend

The frontend is built from the `frontend/` directory:

```yaml
frontend:
  build: ./frontend
  ports:
    - "3000:80"
```

The host exposes the application at:

```text
http://localhost:3000
```

The frontend Dockerfile uses a **multi-stage build**:

```text
Node.js Build Stage
        │
        ├── Install dependencies
        ├── Copy source code
        └── Build application
                │
                ▼
             /app/dist
                │
                ▼
        Nginx Runtime Stage
                │
                ▼
              Port 80
```

This keeps the final runtime focused on serving the built frontend.

---

### Backend

The backend is built from:

```text
./backend
```

and exposes:

```yaml
ports:
  - "5001:5000"
```

This means:

```text
Host:      5001
Container: 5000
```

The backend receives its PostgreSQL configuration through environment variables.

---

### PostgreSQL

The database uses the official Alpine-based PostgreSQL image:

```text
postgres:16-alpine
```

The database runs on the internal Docker network and is accessed by the backend using:

```text
database:5432
```

The database service also uses:

```yaml
restart: always
```

so Docker can automatically restart the database container according to its restart policy.

---

## 💾 Persistent Storage

PostgreSQL uses a named Docker volume:

```yaml
volumes:
  - pgdata:/var/lib/postgresql/data
```

The volume is declared as:

```yaml
volumes:
  pgdata:
```

This separates PostgreSQL's persistent data from the lifecycle of the database container.

```text
PostgreSQL Container
        │
        ▼
/var/lib/postgresql/data
        │
        ▼
Docker Named Volume
       pgdata
```

Because the data is stored in a named volume, recreating the PostgreSQL container does not automatically delete the database contents.

---

## 🔌 Ports

| Service    | Container Port | Host Port | Access   |
| ---------- | -------------: | --------: | -------- |
| Frontend   |           `80` |    `3000` | Browser  |
| Backend    |         `5000` |    `5001` | Host/API |
| PostgreSQL |         `5432` |  Internal | Backend  |

PostgreSQL does not need to be exposed directly to the host because the backend communicates with it through the Docker network.

---

## 🧩 Docker Compose

The complete stack is defined inside:

```text
docker-compose.yml
```

The Compose configuration manages:

```text
frontend
backend
database
```

as well as:

```text
my-custom-network
pgdata
```

### Dependency flow

```text
Frontend
   │
   └── depends_on
          │
          ▼
       Backend
          │
          └── depends_on
                 │
                 ▼
              Database
```

This establishes the expected service dependency order during startup.

---

## 📁 Project Structure

```text
FullStackDocker/
│
├── backend/
│   ├── Dockerfile
│   ├── index.js
│   ├── package.json
│   └── package-lock.json
│
├── frontend/
│   ├── Dockerfile
│   ├── src/
│   ├── public/
│   ├── index.html
│   ├── package.json
│   ├── package-lock.json
│   ├── vite.config.js
│   └── README.md
│
├── docs/
│   └── screenshots/
│       ├── 01-application-running.png
│       ├── 02-docker-compose.png
│       ├── 03-running-containers.png
│       ├── 04-docker-images.png
│       └── 05-frontend-dockerfile.png
│
├── docker-compose.yml
├── .gitignore
└── README.md
```

> `node_modules/` directories should not be committed to the repository.

---

## 🛠️ Tech Stack

| Technology                | Role                           |
| ------------------------- | ------------------------------ |
| **React / Vite**          | Frontend                       |
| **Node.js**               | Backend                        |
| **PostgreSQL 16**         | Relational database            |
| **Docker**                | Containerization               |
| **Docker Compose**        | Multi-container orchestration  |
| **Nginx**                 | Frontend production server     |
| **Docker Bridge Network** | Internal service communication |
| **Docker Volume**         | Persistent database storage    |

---

## ⚙️ Environment Configuration

The backend uses environment variables to connect to PostgreSQL:

```text
DB_HOST=database
DB_USER=postgres
DB_PASS=<database-password>
DB_NAME=postgres
```

The PostgreSQL service uses corresponding configuration variables:

```text
POSTGRES_USER=postgres
POSTGRES_PASSWORD=<database-password>
POSTGRES_DB=postgres
```

### 🔐 Security

For local development, credentials may be provided through environment variables.

For a public repository, secrets should **not** be hardcoded inside `docker-compose.yml`.

Recommended approach:

```text
.env
.env.example
```

The actual `.env` file should be added to `.gitignore`.

Example:

```text
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_local_password
POSTGRES_DB=postgres
```

Commit only the example configuration, never the real credentials.

---

## 🚀 Getting Started

### Prerequisites

Install:

* Docker Desktop
* Git

Verify Docker installation:

```bash
docker --version
```

Verify Docker Compose:

```bash
docker compose version
```

---

### 1. Clone the repository

```bash
git clone https://github.com/connectwithaadi/Docker-and-Kubernetes/tree/main/02%20FullStackDocker-Project
```

Move into the project directory:

```bash
cd FullStackDocker
```

---

### 2. Configure environment variables

Create a local `.env` file if the project uses environment-based credentials.

Example:

```text
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_password
POSTGRES_DB=postgres
```

Do not commit the `.env` file.

---

### 3. Build and start the application

Run:

```bash
docker compose up --build
```

Docker Compose will:

1. Build the frontend image
2. Build the backend image
3. Pull PostgreSQL if required
4. Create the custom network
5. Create the named database volume
6. Start PostgreSQL
7. Start the backend
8. Start the frontend

---

### 4. Open the frontend

Open:

```text
http://localhost:3000
```

The frontend should now be served by the Nginx container.

---

### 5. Access the backend

The backend is mapped to:

```text
http://localhost:5001
```

The exact available endpoints depend on the backend implementation.

---

## 🛑 Stopping the Application

To stop the Compose application:

```bash
docker compose down
```

This removes the containers and network created by Compose while keeping the named volume.

---

## ⚠️ Removing Persistent Database Data

To remove containers **and** the PostgreSQL volume:

```bash
docker compose down -v
```

> ⚠️ This deletes the Docker-managed PostgreSQL data stored in the `pgdata` volume.

Use this command only when you intentionally want to reset the database.

---

## 🔍 Useful Docker Commands

### Check running containers

```bash
docker ps
```

### Check all containers

```bash
docker ps -a
```

### Check Docker images

```bash
docker images
```

### Check volumes

```bash
docker volume ls
```

### Check networks

```bash
docker network ls
```

### Check Compose services

```bash
docker compose ps
```

### View all service logs

```bash
docker compose logs
```

### Follow logs

```bash
docker compose logs -f
```

### View backend logs

```bash
docker compose logs backend
```

### View database logs

```bash
docker compose logs database
```

---

## 🖼️ Screenshots

### 1. Application Running

The frontend application successfully running from the Dockerized environment.

![Application Running](docs/screenshots/01-application-running.png)

---

### 2. Docker Compose Configuration

The Compose configuration showing the frontend, backend, PostgreSQL database, network, and persistent volume.

![Docker Compose](docs/screenshots/02-docker-compose.png)

---

### 3. Running Containers

Docker Desktop showing the running Compose application.

![Running Containers](docs/screenshots/03-running-containers.png)

---

### 4. Docker Images

Docker Desktop showing the PostgreSQL, backend, and frontend images used by the application.

![Docker Images](docs/screenshots/04-docker-images.png)

---

### 5. Frontend Multi-Stage Dockerfile

The frontend Dockerfile uses Node.js for building the application and Nginx as the final runtime server.

![Frontend Dockerfile](docs/screenshots/05-frontend-dockerfile.png)

---


---

## 🎯 Learning Outcomes

By completing this project, I practiced how to:

* Write Dockerfiles
* Build custom Docker images
* Containerize frontend applications
* Containerize backend applications
* Run PostgreSQL using Docker
* Configure Docker Compose
* Create custom Docker networks
* Connect containers through service names
* Configure services with environment variables
* Persist database data using Docker volumes
* Use multi-stage Docker builds
* Serve a frontend through Nginx
* Manage multiple services together
* Inspect containers, images, volumes, and logs
* Troubleshoot a containerized application

---



---



## 👤 Author

**Aditya Kumar Singh**



---

## 📄 License

This project is intended primarily for **learning and portfolio purposes**.

If you choose to make it open source, add an appropriate license such as the MIT License.

