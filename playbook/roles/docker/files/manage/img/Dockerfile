FROM python:3.8-slim-buster as bindep

# set environment variables 
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

COPY ./requirements.txt .
RUN apt-get update && \
    apt-get install -y gcc && \
    pip wheel --no-cache-dir --no-deps --wheel-dir /wheels -r requirements.txt

FROM python:3.8-slim-buster

# set environment variables 
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# installing netcat (nc) since we are using that to listen to postgres server in entrypoint.sh
RUN apt-get update && \
    apt-get install -y --no-install-recommends netcat && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    useradd -u 1500 tank && \
    mkdir /app && \
    chown tank:tank /app

#ARG DOCKER_USER=default_user
# install dependencies
COPY --from=bindep /wheels /wheels
RUN pip install --no-cache /wheels/* && \
    rm -rf /wheels

# copy entrypoint.sh
COPY ./entrypoint.sh /entrypoint.sh

WORKDIR /app
COPY --chown=tank:tank ./django/ .
USER tank
# run entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]