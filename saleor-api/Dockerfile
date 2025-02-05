FROM python:3.12

# Install system dependencies
WORKDIR /app

# Install Poetry
RUN apt-get -y update && apt-get install -y --no-install-recommends \
  gettext \
  libffi8 \
  libgdk-pixbuf2.0-0 \
  liblcms2-2 \
  libopenjp2-7 \
  libssl3 \
  libtiff6 \
  libwebp7 \
  shared-mime-info \
  mime-support \
  libpq5 \
  libpq-dev \
  postgresql-client \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# Install Poetry
RUN pip install poetry==1.8.4

# Disable Poetry virtual environments
RUN poetry config virtualenvs.create false

# Create a non-root user & group for security
RUN groupadd -r saleor && useradd -r -g saleor saleor && chown -R saleor:saleor /app/ 

# Copy dependency files
COPY pyproject.toml poetry.lock /app/ 

# Install dependencies
RUN poetry install --no-root

# Copy application files
COPY . /app/

# Collect static files
ARG STATIC_URL
ENV STATIC_URL=${STATIC_URL:-/static/}
RUN SECRET_KEY=dummy STATIC_URL=${STATIC_URL} python3 manage.py collectstatic --no-input

# Switch to the saleor user before running the application
USER saleor

# Expose the API port
EXPOSE 8000

# Define the command to run the API using Gunicorn with Uvicorn workers
CMD ["gunicorn", "--bind", ":8000", "--workers", "4", "--worker-class", "saleor.asgi.gunicorn_worker.UvicornWorker", "saleor.asgi:application"]