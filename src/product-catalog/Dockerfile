# ==========================================
# STAGE 1: The Builder (Compilation Environment)
# ==========================================
FROM golang:1.22-alpine AS builder

# 1. Install build-essentials if needed (security-patched alpine)
RUN apk add --no-cache ca-certificates git

WORKDIR /usr/src/app

# 2. Optimization: Copy only dependency files first
# This ensures 'go mod download' isn't re-run unless your dependencies change
COPY go.mod go.sum* ./
RUN go mod download

# 3. Copy source code and compile
COPY . .
# -ldflags="-s -w" shrinks the binary by removing debug info (Security/Size win)
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /go/bin/product-catalog ./

# ==========================================
# STAGE 2: The Release (Production Runtime)
# ==========================================
FROM alpine:3.19 AS release

# 4. Security: Add a non-root user to avoid 'Root inside container = Root on Host'
RUN addgroup -S devopsgroup && adduser -S devopsuser -G devopsgroup

# 5. Security: Update Alpine packages to fix known vulnerabilities (CVEs)
RUN apk update && apk upgrade --no-cache

WORKDIR /usr/src/app

# 6. Copy only essential artifacts
COPY ./products ./products
COPY --from=builder /go/bin/product-catalog ./

# 7. Security: Change ownership of the app directory to our non-root user
RUN chown -R devopsuser:devopsgroup /usr/src/app

# 8. Set the running user
USER devopsuser

ENV PRODUCT_CATALOG_PORT 8088

# 9. Exec form ENTRYPOINT for proper signal handling in Kubernetes/EKS
ENTRYPOINT ["./product-catalog"]