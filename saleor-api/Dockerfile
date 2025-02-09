FROM python:3.12

# Set the working directory
WORKDIR /app

# Install system dependencies
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
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Poetry
RUN pip install --no-cache-dir poetry==1.8.4

# Configure Poetry to install dependencies directly in the system environment
RUN poetry config virtualenvs.create false

# Copy dependency files
COPY pyproject.toml poetry.lock /app/

# Install dependencies
RUN poetry install --no-root

# Create a non-root user & group for security
RUN groupadd -r saleor && useradd -r -g saleor saleor && chown -R saleor:saleor /app/

# Switch to the saleor user before copying application files
USER saleor

# Copy application files
COPY --chown=saleor:saleor . /app/

# Collect static files
ARG STATIC_URL
ENV STATIC_URL=${STATIC_URL:-/static/}
RUN SECRET_KEY=dummy STATIC_URL=${STATIC_URL} python3 manage.py collectstatic --no-input

# Expose the application port
EXPOSE 8000

# Define the command to run the API using Gunicorn with Uvicorn workers
CMD ["gunicorn", "--bind", ":8000", "--workers", "4", "--worker-class", "saleor.asgi.gunicorn_worker.UvicornWorker", "saleor.asgi:application"]
