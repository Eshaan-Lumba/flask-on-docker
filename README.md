# Flask on Docker

## Overview

In this repository, we develop a Flask application to run using Docker. The application supports basic static webpages as well as allowing the user to upload an image and view their uploaded images. Postgres was used for the database, and Nginx and Gunicorn were included to support the production environment in order to help handle the static and media file upload/view functionalities. Docker allowed us to containerize the application, and we built separate Dockerfiles for both development and production environments. This project was inspired from a [tutorial](https://testdriven.io/blog/dockerizing-flask-with-postgres-gunicorn-and-nginx/).

Below, we can see an example demo of uploading an image. 

![Upload demo](upload-demo.gif)

## Build Instructions

Note that to run the following commands, you need to install `docker-compose`. Successfully running 
```
$ pip3 install docker-compose
```
will do it for you. 

### Development

First, we can use `docker-compose` to build the Docker image and run the container.
```
$ docker-compose up -d --build
```
We now need to start and create the databases:
```
$ docker-compose exec web python manage.py create_db
```
To test, we can access [http://localhost:1365](http://localhost:1365) and see output similar to below:
```
$ curl http://localhost:1365
{
  "hello": "world"
}
```
> **Aside:**
> In the above demo, you might note that I am accessing [http://localhost:8080](http://localhost:8080). I enabled this using port forwarding as I was working on a lambda server. For normal testing, [http://localhost:1365](http://localhost:1365) is the correct endpoint.

To confirm that the table was successfully created:
```
$ docker-compose exec db psql --username=hello_flask --dbname=hello_flask_dev

hello_flask_dev=# \l
```
and you should see the `hello_flask_dev` table.

If we access [http://localhost:1365/static/hello.txt](http://localhost:1365/static/hello.txt), we can see the following output:
```
$ curl http://localhost:1365/static/hello.txt
hi!
```
In order to upload an image, access [http://localhost:1365/upload](http://localhost:1365/upload).

To view the image uploaded, access: [http://localhost:1365/media/IMAGE_FILE_NAME](http://localhost:1365/media/IMAGE_FILE_NAME).

Finally, we can bring down the development containers (the `-v` flag also brings down the associated volumes):
```
$ docker-compose down -v
```

### Production

Note that this repository does not contain the `.env.prod` and `.env.prod.db` environment variable files that are necessary for the services to run from the container for security purposes. 

Again, we need to build the production Docker image and run the container. 
```
$ docker-compose -f docker-compose.prod.yml up -d --build
```
Similar to above, we create the table:
```
$ docker-compose -f docker-compose.prod.yml exec web python manage.py create_db
```
To test, we can access [http://localhost:1365](http://localhost:1365) and see output similar to below:
```
$ curl http://localhost:1365
{"hello": "world"}
```
If we access [http://localhost:1365/static/hello.txt](http://localhost:1365/static/hello.txt)`, we can see the following output:
```
$ curl http://localhost:1365/static/hello.txt
hi!
```
In order to upload an image, access [http://localhost:1365/upload](http://localhost:1365/upload).

To view the image uploaded, access: [http://localhost:1365/media/IMAGE_FILE_NAME](http://localhost:1365/media/IMAGE_FILE_NAME).

Finally, we can bring down the containers:
```
$ docker-compose -f docker-compose.prod.yml down -v
```
