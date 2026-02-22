# ==========================================
# STAGE 1: Builder (The Industrial Kitchen)
# ==========================================
FROM python:3.12-slim-bookworm AS builder

WORKDIR /usr/src/app/

# Install build dependencies needed for psutil
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    python3-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .

# Create "Wheels" (compiled binaries) for all requirements
RUN pip install --upgrade pip && \
    pip wheel --no-cache-dir --no-deps --wheel-dir /usr/src/app/wheels -r requirements.txt

# ==========================================
# STAGE 2: Runtime (The Clean Room)
# ==========================================
FROM python:3.12-slim-bookworm

WORKDIR /usr/src/app/

# Copy the compiled wheels from the builder
COPY --from=builder /usr/src/app/wheels /wheels
COPY requirements.txt .

# Install the pre-compiled wheels (No GCC needed here!)
RUN pip install --upgrade pip && \
    pip install --no-cache /wheels/*

COPY . .

# DevSecOps: Run as non-root (Optional but recommended)
# RUN useradd -m devopsuser
# USER devopsuser

ENTRYPOINT ["python", "recommendation_server.py"]