# App

A minimal Go web server with two HTTP routes.

## Routes

| Route | Method | Response |
|---|---|---|
| `/` | GET | HTML: `<h1>Welcome to the TP CI/CD Go Application!</h1>` |
| `/health` | GET | JSON: `{"status":"OK"}` |

## Run locally

Install tools first:

```bash
mise install
```

Start the server:

```bash
cd app && go run main.go
# Listening on :8080
```

Test the endpoints:

```bash
curl http://localhost:8080/
curl http://localhost:8080/health
```

## Lint & Test

```bash
mise run lint   # golangci-lint run ./...
mise run test   # go test ./...
```

## Docker

Build the image locally:

```bash
docker build -t tp-ci-cd:local ./app
```

The image uses a multi-stage build: compiled in `golang:1.21-alpine`, runs in `alpine:3.19.1` as a non-root user. CGO is disabled and debug symbols are stripped to minimize image size.
