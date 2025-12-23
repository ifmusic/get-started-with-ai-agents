# syntax=docker/dockerfile:1

# --- Stage 1: Build the Vite Frontend ---
FROM node:22-bookworm-slim AS frontend-builder
RUN npm install -g pnpm@10.6.0
WORKDIR /app/frontend

COPY frontend/package.json frontend/pnpm-lock.yaml* ./
RUN pnpm install --frozen-lockfile=false

COPY frontend/ ./
# This command creates files in /app/api/static/react (based on your logs)
RUN pnpm build

# --- Stage 2: Final Python Runtime ---
FROM python:3.13.8-slim-bookworm
WORKDIR /code

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip>=25.3 \
    && pip install --no-cache-dir -r requirements.txt

# Copy all backend code first
COPY . .

# COPY FROM THE CUSTOM VITE PATH
# Based on your log: ../api/static/react/
# In the builder stage, this resolves to /app/api/static/react/
COPY --from=frontend-builder /app/api/static/react/ ./api/static/react/

# Security cleanup
RUN apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/*

EXPOSE 50505
CMD ["gunicorn", "api.main:create_app"]
