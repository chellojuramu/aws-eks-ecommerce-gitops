# ==========================================
# STAGE 1: Builder (The Industrial Kitchen)
# ==========================================
FROM eclipse-temurin:21-jdk AS builder

WORKDIR /usr/src/app/

# 1. Optimization: Copy only build-definition files first
# This allows Docker to cache the 'download' layer
COPY gradlew* settings.gradle* build.gradle ./
COPY gradle/ ./gradle/

# 2. Grant execution permission to the Gradle wrapper
RUN chmod +x ./gradlew

# 3. Download dependencies (This layer is cached unless build.gradle changes)
RUN ./gradlew downloadRepos

# 4. Copy the actual source code and Proto files
COPY . .
COPY ./pb ./proto

# 5. Build the application
# 'installDist' creates a clean folder with just the runnable app
RUN ./gradlew installDist -PprotoSourceDir=./proto

# ==========================================
# STAGE 2: Runtime (The Clean Room)
# ==========================================
# We use JRE instead of JDK to reduce the attack surface (No compilers here!)
FROM eclipse-temurin:21-jre

# DevSecOps: Create a non-root system user and group (Ubuntu/Debian syntax)
RUN addgroup --system devopsgroup && \
    adduser --system --ingroup devopsgroup devopsgroup

WORKDIR /usr/src/app/

# 6. Optimization: Copy ONLY the compiled app from the builder
# We don't need the source code or Gradle anymore!
COPY --from=builder /usr/src/app/build/install/opentelemetry-demo-ad/ ./

# 7. Security: Ownership transfer to non-root user
RUN chown -R devopsuser:devopsgroup /usr/src/app/

# Set the active user
USER devopsuser

# Cloud-Native Config: Default port that can be overridden by K8s ConfigMaps
ENV AD_PORT=9099

# Exec form entrypoint for proper SIGTERM signal handling in Kubernetes
ENTRYPOINT ["./bin/Ad"]