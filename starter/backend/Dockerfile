FROM public.ecr.aws/docker/library/python:3.10-slim
ENV FLASK_RUN_HOST=0.0.0.0

WORKDIR /app

RUN useradd -u 1000 -m app && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
      build-essential \
      gcc \
      libpcre3 \
      libpcre3-dev \
      libssl-dev \
      libffi-dev \
    && pip install --no-cache-dir pipenv uwsgi \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

COPY Pipfile* /app/
RUN pipenv install --system --ignore-pipfile

COPY . /app

CMD ["uwsgi", "--http", ":5000", "--master", "--enable-threads", "--manage-script-name", "--mount", "/=app:app"]