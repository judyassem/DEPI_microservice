# DEPI Microservice: Multi-Container App with Docker Compose and CI/CD
 
A multi-container application made of a React client, a Node.js API, a background worker, PostgreSQL, Redis and an Nginx reverse proxy. The whole stack is defined in Docker Compose and starts with a single command. A GitHub Actions workflow automates the CI/CD pipeline.
 
> Built as part of the DevOps track of the Digital Egypt Pioneers Initiative (DEPI).
 
## Architecture
 
```
                 ┌──────────┐
   Browser ────► │  Nginx   │  (port 3050)
                 └────┬─────┘
            ┌─────────┴─────────┐
            ▼                   ▼
       ┌─────────┐         ┌─────────┐
       │ client  │         │   api   │
       │ (React) │         │ (Node)  │
       └─────────┘         └───┬─────┘
                        ┌──────┴──────┐
                        ▼             ▼
                  ┌──────────┐   ┌─────────┐        ┌──────────┐
                  │ Postgres │   │  Redis  │ ◄───── │  worker  │
                  └──────────┘   └─────────┘        └──────────┘
```
 
| Service | Role | Built from |
|---|---|---|
| `nginx` | Reverse proxy; single entry point routing traffic to the client and the API | `./nginx` |
| `client` | React front end | `./client` |
| `api` | Node.js backend; connects to PostgreSQL and Redis | `./server` |
| `worker` | Background process connected to Redis | `./worker` |
| `postgres` | Relational database | `postgres:latest` image |
| `redis` | In-memory data store / message layer | `redis:latest` image |
 
## Docker Compose highlights
 
- Five services are built from their own `Dockerfile.dev`; PostgreSQL and Redis use official images.
- Services find each other by name on the Compose network (for example `PGHOST=postgres`, `REDIS_HOST=redis`).
- Configuration is passed through environment variables.
- Bind mounts (`./server:/app`, `./client:/app`, `./worker:/app`) give live code reloading during development, while an anonymous volume on `/app/node_modules` keeps the container's dependencies intact.
- `depends_on` sets the startup order for Nginx, and `restart: always` keeps the proxy running.
## Run locally
 
Requirements: Docker and Docker Compose.
 
```bash
git clone https://github.com/judyassem/DEPI_microservice.git
cd DEPI_microservice
docker compose up --build
```
 
Open <http://localhost:3050>.
 
Stop and clean up:
 
```bash
docker compose down
```
 
## CI/CD
 
The pipeline is defined with GitHub Actions in `.github/workflows/`. See the workflow file there for the exact steps.
 
## Repository layout
 
```
.github/workflows/   CI/CD pipeline
client/              React front end
server/              Node.js API
worker/              background worker
nginx/               reverse proxy configuration
docker-compose.yaml  full stack definition
```
 
## Tech stack
 
Docker, Docker Compose, GitHub Actions, Nginx, Node.js, React, PostgreSQL, Redis
 
## Notes
 
The database password in `docker-compose.yaml` is a development placeholder. For any real deployment, move credentials to an `.env` file or a secrets manager and keep them out of version control.
