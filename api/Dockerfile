FROM python:3.8.2
LABEL maintainer twtrubiks
ENV PYTHONUNBUFFERED 1
RUN mkdir /docker_api
WORKDIR /docker_api
COPY . /docker_api
RUN pip install -r requirements.txt
# RUN pip install  -i  https://pypi.python.org/simple/  -r requirements.txt

# for entry point
RUN chmod +x /docker_api/docker-entrypoint.sh