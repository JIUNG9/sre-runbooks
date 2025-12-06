# Multi-Stage Docker Builds

## Go example
```dockerfile
# Build stage
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app/server

# Runtime stage
FROM gcr.io/distroless/static-debian12
COPY --from=builder /app/server /server
USER nonroot:nonroot
ENTRYPOINT ["/server"]
```

## Tips
- Use distroless or scratch for minimal attack surface
- Pin base image digests in production
- Layer caching: copy dependency files first
- Use .dockerignore aggressively
