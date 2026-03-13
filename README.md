# go-hello

A simple Go HTTP server with Docker and docker-compose setup, including PostgreSQL and Redis services.

## Quick Start

### Prerequisites

- Docker ([install](https://docs.docker.com/get-docker/))
- Docker Compose (included with Docker Desktop)
- Go 1.21+ (optional, for local development)

## Building

### Build the Docker Image

```bash
docker build -t go-hello .
```

This uses a multi-stage build to create a minimal Alpine Linux image:

1. **Builder stage**: Compiles Go code with CGO disabled for cross-platform compatibility
2. **Final stage**: Copies only the compiled binary into a minimal Alpine image (~15MB)

### Build with docker-compose

When using docker-compose, the image is automatically built:

```bash
docker compose up -d
```

## Running

### Run Standalone Container

```bash
# Run the server
docker run -p 8080:8080 go-hello

# In another terminal, test it
curl http://localhost:8080
```

Expected output: `Hello World from Go!`

### Run with docker-compose (Recommended)

```bash
# Start all services
docker compose up -d

# View logs
docker compose logs -f go-hello

# Stop all services
docker compose down
```

To remove volumes as well:

```bash
docker compose down -v
```

## Services

### go-hello

- **Image**: Built from local Dockerfile
- **Port**: `8080`
- **Purpose**: HTTP server that responds with "Hello World"
- **Environment Variables**:
  - `DB_HOST=postgres` — PostgreSQL hostname
  - `DB_PORT=5432` — PostgreSQL port
  - `DB_USER=postgres` — PostgreSQL user
  - `DB_PASSWORD=postgres` — PostgreSQL password
  - `DB_NAME=myapp` — PostgreSQL database name
  - `REDIS_HOST=redis` — Redis hostname
  - `REDIS_PORT=6379` — Redis port

### postgres

- **Image**: `postgres:16`
- **Port**: `5432`
- **Purpose**: Relational database for application data
- **Environment Variables**:
  - `POSTGRES_USER=postgres`
  - `POSTGRES_PASSWORD=postgres`
  - `POSTGRES_DB=myapp`
- **Volume**: `postgres_data` (persists data between container restarts)
- **Health Check**: Verifies connection before go-hello starts

### redis

- **Image**: `redis:7`
- **Port**: `6379`
- **Purpose**: In-memory key-value store for caching and session management
- **Health Check**: Verifies readiness (ping command)

## Testing Connectivity

All services are on the same Docker network (`app-network`) and can reach each other by hostname:

```bash
# Test go-hello
curl http://localhost:8080

# Test PostgreSQL
docker compose exec postgres psql -U postgres -c "SELECT 1;"

# Test Redis
docker compose exec redis redis-cli ping
```

## Pushing to Docker Hub

Tag the image with your Docker Hub username:

```bash
docker tag go-hello yourusername/go-hello
```

Log in and push:

```bash
docker login
docker push yourusername/go-hello
```

For this project, the image is pushed to:
```

docker push aldino97/go-hello
```

## Project Structure

```

go-hello/
├── main.go              # Go HTTP server source code
├── go.mod               # Go module definition
├── Dockerfile           # Multi-stage Docker build
├── docker-compose.yml   # Service orchestration (go-hello, postgres, redis)
└── README.md            # This file
```

## Development

### Run Locally Without Docker

```bash
go run main.go
```

Server starts on `localhost:8080`

### Modify the Server

Edit [main.go](main.go) to change the HTTP handler or add routes:

```go
func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello World from Go!\n")
}
```

## Troubleshooting

**Container exits immediately:**

- Check logs: `docker compose logs go-hello`

**Port 8080 already in use:**

- Modify docker-compose.yml: change `"8080:8080"` to `"8081:8080"`

**PostgreSQL connection refused:**

- Wait for health check: `docker compose ps` shows `(health: healthy)`

**Cannot reach services by hostname:**

- Ensure all containers are on the same network: `docker network ls`
- Verify DNS: `docker compose exec go-hello nslookup postgres`

## Notes

- The Dockerfile uses Alpine Linux (minimal, secure base image)
- `CGO_ENABLED=0` ensures the binary doesn't depend on system C libraries
- Health checks prevent go-hello from starting before databases are ready
- Volumes persist PostgreSQL data across container restarts
