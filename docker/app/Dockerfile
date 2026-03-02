# docker/app/Dockerfile
FROM python:3.13-slim-trixie
EXPOSE 8080

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Set work directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*


# --- Tarea 1: Refuerzo del contenedor ---
# Crear el usuario específico sin privilegios
RUN useradd -m privuser 

# Cambiar la propiedad del directorio de trabajo
RUN chown -R privuser /app    
    

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
RUN pip install --no-cache-dir uvicorn

# Copy project
COPY sportsclub/ .

# Collect static files (needed when serving through a reverse proxy)
RUN python manage.py collectstatic --noinput

# --- Tarea 1: Asegurar que la aplicación se ejecute con este usuario ---
USER privuser


# Run uvicorn
CMD ["uvicorn", "sportsclub.asgi:application", "--host", "0.0.0.0", "--port", "8080"]
