FROM public.ecr.aws/docker/library/python:3.10-slim
ENV FLASK_RUN_HOST=0.0.0.0

WORKDIR /app

# Install system build deps needed for uwsgi
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential \
        libpcre3 \
        libpcre3-dev && \
    rm -rf /var/lib/apt/lists/*

# Install all Python dependencies directly with pip (no pipenv lock file issues)
RUN pip install --no-cache-dir \
        flask \
        flask-cors \
        uwsgi \
        pytest

# Copy application source
COPY . /app

# Expose the app on port 5000
EXPOSE 5000

CMD ["uwsgi", "--http", ":5000", "--master", "--enable-threads", "--manage-script-name", "--mount", "/=app:app"]