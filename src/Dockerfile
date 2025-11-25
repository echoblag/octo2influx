FROM python:3.12-slim-bookworm

LABEL maintainer="Blag Ivanov <88674890+echoblag@users.noreply.github.com>"

WORKDIR /usr/src/app

RUN useradd -u 9923 octo2influx --create-home

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY octo2influx.py loop.sh ./

RUN chown 9923:9923 /usr/src/app/*

USER octo2influx

ENV FREQ=1h
ENTRYPOINT [ "bash", "loop.sh" ]
