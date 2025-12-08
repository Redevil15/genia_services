# === Etapa 1: Builder ===
FROM python:3.11-slim as builder

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /build

COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# === Etapa 2: Runtime ===
FROM python:3.11-slim

LABEL maintainer="brandonf2345@gmail.com" \
      service="credit-scoring" \
      version="1.0.0"

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app \
    # AWS ECS inyecta PORT, default 8000 para local
    PORT=8000

WORKDIR /app

# Usuario no-root (seguridad)
RUN groupadd --gid 1000 appgroup && \
    useradd --uid 1000 --gid appgroup --shell /bin/bash appuser

# Copiar dependencias
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin

# Copiar solo lo necesario
COPY src/ /app/src/
COPY models/ /app/models/

RUN chown -R appuser:appgroup /app
USER appuser

# Health check para ALB/ECS
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen(f'http://localhost:{os.environ.get(\"PORT\", 8000)}/health')" || exit 1

EXPOSE ${PORT}

# Producción: usar $PORT (ECS lo inyecta)
CMD ["sh", "-c", "uvicorn src.server.app:app --host 0.0.0.0 --port ${PORT:-8000} --workers 2"]