FROM public.ecr.aws/docker/library/python:3.10-slim
ENV FLASK_RUN_HOST=0.0.0.0
ENV PYTHONPATH=/

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential \
        libpcre3 \
        libpcre3-dev && \
    rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir \
        flask \
        flask-cors \
        uwsgi \
        pytest

WORKDIR /

COPY . /app

EXPOSE 5000

CMD ["uwsgi", "--http", ":5000", "--master", "--enable-threads", "--manage-script-name", "--mount", "/=app:app"]