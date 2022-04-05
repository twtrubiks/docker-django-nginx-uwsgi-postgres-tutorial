# docker-django-nginx-uwsgi-postgres-tutorial

 Docker + Django + Nginx + uWSGI + Postgres Basic Tutorial - from nothing to something

 This tutorial teaches you how to setup [Django](https://www.djangoproject.com/) + [Nginx](https://nginx.org/en/) + [uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/) + [PostgreSQL](https://www.postgresql.org/) with Docker 📝

For those who are not familiar with [Docker](https://www.docker.com/), I suggest that you read my previous tutorials first:

[Docker 基本教學 - 從無到有 Docker-Beginners-Guide 教你用 Docker 建立 Django + PostgreSQL 📝](https://github.com/twtrubiks/docker-tutorial)

* [Youtube Tutorial PART 1 - Docker + Django + Nginx + uWSGI + Postgres - 簡介](https://youtu.be/u4XIMTOsxJk)
* [Youtube Tutorial PART 2 - Docker + Django + Nginx + uWSGI + Postgres - 原理步驟](https://youtu.be/9K4O1UuaXrU)
* [Youtube Tutorial PART 3 - Docker + Django + Nginx + uWSGI + Postgres - 實戰](https://youtu.be/v7Mf9TuROnc)

## Summary

### [Docker](https://www.docker.com/)

![](https://i.imgur.com/gDcSwcs.png)

I have introduced Docker before, so I won't introduce it here :stuck_out_tongue_closed_eyes:

Take a look at:

[Docker 基本教學 - 從無到有 Docker-Beginners-Guide 教你用 Docker 建立 Django + PostgreSQL 📝](https://github.com/twtrubiks/docker-tutorial)

### [Django](https://github.com/django/django)

Take a look at:

[Django 基本教學 - 從無到有 Django-Beginners-Guide 📝](https://github.com/twtrubiks/django-tutorial)

[Django-REST-framework 基本教學 - 從無到有 DRF-Beginners-Guide 📝](https://github.com/twtrubiks/django-rest-framework-tutorial)

For more Django examples, check out my [Github](https://github.com/twtrubiks?utf8=%E2%9C%93&tab=repositories&q=Django&type=&language=), I have just listed two of the simpler ones here :relaxed:

### [PostgreSQL](https://www.postgresql.org/)

![](https://i.imgur.com/RrNtbfz.png)

### [Nginx](https://nginx.org/en/)

![](https://i.imgur.com/AkcCtDa.png)

Nginx is a type of Web Server that uses few resources and has high stability.

Nginx's high stability is a result of solving the **C10K** problem. What is **C10K**? The original literature can be found here [The C10K problem](http://www.kegel.com/c10k.html).

**C10K** is the Client 10000 problem. Previously, when a server receives more than 10000 concurrent connections, it may not be able to operate normally.

Nginx does not natively support dynamic content, so, any dynamic content needs to be set up separately. [uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/) is used to facilitate the communication between Nginx and the dynamic content.

Take a look at the diagram below (important)

:star: the web client <-> the web server ( Nginx )  <-> unix socket  <-> uWSGI <-> Django :star:

You may ask me, what is uWSGI :confused:?

uWSGI implements a communication protocol. You can think of it as a connector (which communicates with Django).

Usually, Django will be put behind the http server ( Nginx ), so, when the server receives a request, how will it pass the data to Django?


This is uWSGI's functionality :wink:

So why do we still need Nginx :confused:?

First, let's understand a concept,

Nginx is in charge of static content (html, js, css, images, ...), uWSGI is in charge of Python's dynamic content.

uWSGI does not handle static content well (it's inefficient), so, we can use Nginx to handle static content. In addition, Nginx has a lot of other benefits.

* Nginx, compared to uWSGI, handles static resources better
* Nginx allows cache configuration
* Nginx can act as a reverse proxy
* Nginx can load balance multiple connections

Gentle reminder :heart:

If you would like learn more about **reverse proxies**, you can take a look at [forward proxy VS reverse proxy](https://github.com/twtrubiks/docker-django-nginx-uwsgi-postgres-load-balance-tutorial#%E6%AD%A3%E5%90%91%E4%BB%A3%E7%90%86%E5%99%A8--vs-%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E5%99%A8) :smile:

Up to this point, you may have some questions:

Why can I still run Django without Nginx and uWSGI? :confused:

To run Django, we usually use `python manage.py runserver` to run the server.

Actually, when you run this command, Django helps you start a small http server.

Of course, this is just for convenience during development. We would not do this in production ( not to mention performance :disappointed_relieved: )

Hey :confused: isn't there Gunicorn? Previously you mentioned Gunicorn in
[Deploying_Django_To_Heroku_Tutorial](https://github.com/twtrubiks/Deploying_Django_To_Heroku_Tutorial)
[Deploying-Flask-To-Heroku](https://github.com/twtrubiks/Deploying-Flask-To-Heroku)

So why can't we use Gunicorn instead of uWSGI?

The last time, when I used Gunicorn, it was because the application resided in Heroku, and it is recommended to use Gunicorn to set up a web server there. As for which is better, Gunicorn or uWSGI, I feel that it is up your use case :wink:

Wait a minute, since we are talking about Nginx, isn't there also [Apache](https://httpd.apache.org/)? I heard that a lot of people use that :stuck_out_tongue_winking_eye:

You may also be asking, so should I choose [Nginx](https://nginx.org/en/) or [Apache](https://httpd.apache.org/) :confused:

I think that there is no best server, if the server meets your requirements, the choose it :smiley:

## Tutorial

This time, I will be using Docker to set up 3 containers to separate Nginx, Django + uWSGI and Postgres

My main reference is [https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html](https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html) in this tutorial.

But there are some differences :smirk:

The main focus this time will be on setting up of Nginx with Django + uWSGI

**Nginx Section**, you can take a look at [Dockerfile](https://github.com/twtrubiks/docker-django-nginx-uswgi-postgres-tutorial/blob/master/nginx/Dockerfile) in the folder.

```Dockerfile
FROM nginx:latest

COPY nginx.conf /etc/nginx/nginx.conf
COPY my_nginx.conf /etc/nginx/sites-available/

RUN mkdir -p /etc/nginx/sites-enabled/\
    && ln -s /etc/nginx/sites-available/my_nginx.conf /etc/nginx/sites-enabled/

# RUN mkdir -p /etc/nginx/sites-enabled/\
#     && ln -s /etc/nginx/sites-available/my_nginx.conf /etc/nginx/sites-enabled/\
#     && rm /etc/nginx/conf.d/default.conf

CMD ["nginx", "-g", "daemon off;"]
```

Let me explain the steps,

First Step

Copy nginx.conf to the `/etc/nginx/nginx.conf` path.

(the original nginx.conf can be retrieved from the Nginx Docker container, inside the `/etc/nginx` path you can find nginx.conf)

I have copied out a part of the original file [nginx_origin.conf](https://github.com/twtrubiks/docker-django-nginx-uswgi-postgres-tutorial/blob/master/nginx/nginx_origin.conf) :smiley:

For nginx.conf, there are mainly two parts that needs to be modified.

The first part is to change the user to root:

```conf
user  root;
```

The other part is:

```conf
# include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-available/*;
```

Add a line `include /etc/nginx/sites-available/*;`

and remove `include /etc/nginx/conf.d/*.conf;`

This way the [Dockerfile](https://github.com/twtrubiks/docker-django-nginx-uswgi-postgres-tutorial/blob/master/nginx/Dockerfile)
in the Nginx folder does not need the command to delete the default.conf because `include /etc/nginx/conf.d/*.conf;` is the default page that will run.

But now, we need to configure it :smirk:

Second Step

Copy my_nginx.conf into `/etc/nginx/sites-available/`

Let us pause here for a while,

If you use `FROM nginx:latest` to install Nginx, you will discover that you do not have the two directories below:

`/etc/nginx/sites-available/`

`/etc/nginx/sites-enabled/`

but don't worry, if the directories aren't there, we will add them in ( which is what the commands in the Nginx Dockerfile is doing )

But why don't we have these directories in the first place :confused:

The reason is because these directories are only created when Nginx is installed with `apt-get`.

Third Step

sites-available This directory is actually not important, you can create a folder with a name you want too, but

*sites-enabled* This directory is more important because we need to use symlinks (through the Linux command `ln`) link *sites-enabled* with *my_nginx.conf.

Next, let's talk about the configuration in my_nginx.conf

```conf
# the upstream component nginx needs to connect to
upstream uwsgi {
    # server api:8001; # use TCP
    server unix:/docker_api/app.sock; # for a file socket
}

# configuration of the server
server {
    # the port your site will be served on
    listen    80;
    # index  index.html;
    # the domain name it will serve for
    # substitute your machine's IP address or FQDN
    server_name  twtrubiks.com www.twtrubiks.com;
    charset     utf-8;

    client_max_body_size 75M;   # adjust to taste

    # Django media
    # location /media  {
    #     alias /docker_api/static/media;  # your Django project's media files - amend as required
    # }

    location /static {
        alias /docker_api/static; # your Django project's static files - amend as required
    }

    location / {
        uwsgi_pass  uwsgi;
        include     /etc/nginx/uwsgi_params; # the uwsgi_params file you installed
    }

}
```

Let's first look at the upstream part.

It uses Unix sockets which is better than using TCP port sockets because there is less overhead.

Next is `include     /etc/nginx/uwsgi_params`, usually the uwsgi_params can be found in the Nginx directory `/etc/nginx`.

If you really can't find it, you can copy one from [uwsgi_params](https://github.com/twtrubiks/docker-django-nginx-uswgi-postgres-tutorial/blob/master/api/uwsgi_params) (I copied out the file for everyone but if you follow my steps, you will probably have it)

Next, let's talk about uwsgi_pass or what you have seen is proxy_pass.

Nginx will convert the request received to uwsgi's protocol. Then, it will pass the request to Django to handle it.

Then why can't we just use a proxy ( default to http protocol ) but we have to use uwsgi :confused:?

The main reason is because efficiency is taken into consideration.

Since we have already talked about it so much, I'll briefly explain what is a **Proxy server**.

When a client from an external network sends a request, the proxy server will forward it to an internal server to handle it. Once it's done, the response goes back through the proxy server to the client in the external network.

What's the benefit in this :confused:? The benefit is that it protects the internal server, preventing clients from directly attacking the internal server.

Another benefit is the caching mechanism. If the client accesses similar resources, the resources can be retrieved directly from cache.

Last Step

Gentle reminder :heart:

What is a daemon :question::question::question:

Actually, it is not very hard to understand, you can think of it as a type of service :smile:

If you want to learn more about daemons, you can google **linux daemon** :pencil2:

Why do we use `nginx -g daemon off` to start Nginx but not `/etc/init.d/nginx start` :confused:?

This question requires us to go back and understand Docker.

The excerpt below is from [Docker Nginx](https://hub.docker.com/_/nginx/)

***If you add a custom CMD in the Dockerfile, be sure to include -g daemon off; in the CMD in order for nginx to stay in the foreground, so that Docker can track the process properly (otherwise your container will stop immediately after starting)!***

Simply put, it is to keep the Nginx service running. If not, the container will exit.

**Django + uWSGI Section**, you can take a look at the [Dockerfile](https://github.com/twtrubiks/docker-django-nginx-uswgi-postgres-tutorial/blob/master/api/Dockerfile) in the api folder

It it mostly quite simple, but there I would like to point out one thing:

Sometimes when we `pip install`, it runs very slowly.

In these situations, we can add a `-i` option to change the index source, making it run a bit faster :grin:

Next, I'll explain uwsgi.ini, inside of which are some configuration files

```ini
[uwsgi]

# http=0.0.0.0:8000
socket=app.sock
master=true
# maximum number of worker processes
processes=4
threads=2
# Django's wsgi file
module=django_rest_framework_tutorial.wsgi:application

# chmod-socket=664
# uid=www-data
# gid=www-data

# clear environment on exit
vacuum          = true
```

Communication to Nginx is done through the socket file ( app.sock ). The uid and gid parts are permissions.

You can take a look at the article below. The article includes why we do not use root permissions.

[Things to know (best practices and 「issues」) READ IT !!!](http://uwsgi-docs.readthedocs.io/en/latest/ThingsToKnow.html)

I decided to use root anyways because without root, there will be permission errors which I finally found my answer [here](https://stackoverflow.com/questions/18480201/ubuntu-nginx-emerg-bind-to-0-0-0-080-failed-13-permission-denied)

*the socket API bind() to a port less than 1024, such as 80 as your title mentioned, need root access.*

The simpler solution is to run with root :smile:

Lastly, I used `docker-compose.yml` to manage my containers.

See my [docker-compose.yml](https://github.com/twtrubiks/docker-django-nginx-uswgi-postgres-tutorial/blob/master/docker-compose.yml)

## Steps to run

Run `docker-compose up` and watch the magic happen.

You will see something like this:

![](https://i.imgur.com/4WPac2V.png)

![](https://i.imgur.com/I67WDJU.png)

If you then see something like the below, then it has run successfully.

![](https://i.imgur.com/WwRLm4C.png)

![](https://i.imgur.com/G28IGca.png)

Next, go to [http://localhost:8080/api/music/](http://localhost:8080/api/music/)

If you immediately see something like the image below, it means that you have completed a small step.

Seeing this is normal, because we still need to migrate.

![](https://i.imgur.com/d0jlMo9.png)

The terminal output is also fine ( though it is easy to get stuck here :sweat_smile: )

![](https://i.imgur.com/RBW8eQt.png)

Next, open another terminal and go into the api ( Django + uWSGI ) container.

You can find the steps in the previous [docker-tutorial-指令介紹](https://github.com/twtrubiks/docker-tutorial#指令介紹)，

You can also use other GUI applications like the [previously introduced portainer](https://github.com/twtrubiks/docker-tutorial#其他管理-docker-gui-的工具)

```cmd
docker exec -it <Container ID> bash
```

```cmd
python manage.py makemigrations musics
python manage.py migrate
python manage.py createsuperuser
```

![](https://i.imgur.com/haHcokf.png)

This time, we need to run a different command

```cmd
python manage.py collectstatic
```

This takes all the static files in Django and collates them into the static folder.

![](https://i.imgur.com/zaz2bYX.png)

Next, you can go to [http://localhost:8080/api/music/](http://localhost:8080/api/music/), then you will see the normal page :smile:

![](https://i.imgur.com/EybsFZ3.png)

Why do we need to do this step?

The purpose of the step is to pass all the static content to Nginx to process. In my_nginx.conf, you will notice that we pointed Nginx to the `/docker_api/static` path.

As said above, Nginx is in charge of all the static content ( html, css, images, ...... ) while uWSGI is in charge of Python's dynamic content.

If you are interested to try it out, use Django + uWSGI without Nginx. If you do this, it will function normally but you will notice that the css, images and others cannot be retrieved, as shown below

![](https://i.imgur.com/btiI68s.png)

Because uWSGI does not handle static content well :sob:

Though this can be resolved. Take a look at [https://uwsgi-docs.readthedocs.io/en/latest/StaticFiles.html](https://uwsgi-docs.readthedocs.io/en/latest/StaticFiles.html)

but I recommend that you use Nginx, because it can do more things :smiley:

## Final Result

Go to [http://localhost:8080/api/music/](http://localhost:8080/api/music/)

![](https://i.imgur.com/jl43jST.png)

![](https://i.imgur.com/Fw6LjbE.png)

## `hosts` Configuration and Finding Your IP Address

Edit the `hosts` configuration file

Windows

`hosts` is located at

> C:\WINDOWS\system32\drivers\etc\hosts

![](https://i.imgur.com/Q6lZyK0.png)

You may need permissions to save the file.

MAC

`hosts` is located at

> sudo vi /etc/hosts

Finding your IP address

Windows

```cmd
ipconfig
```

![](https://i.imgur.com/sPOqIxM.png)

MAC

```cmd
ifconfig
```

![](https://i.imgur.com/BOs5BwZ.png)

Let's say your ip address is 192.168.1.103, then those in the same network as you (intranet), can connect to you through this ip address.

As for the editing the `hosts` configuration file part, we can just go to [http://twtrubiks.com:8080/api/music/](http://twtrubiks.com:8080/api/music/)

![](https://i.imgur.com/efEqLd0.png)

## Introducing Supervisor

[Supervisor](http://supervisord.org/)

is a type of process management tool. Using it, you can easily start, stop, restart and monitor one or many processes.

For example, if a process stalls, once Supervisor notices, it will automatically restart the process. No need to write any programs ( such as your own shell programs ) to control it.

Now, you may ask me:

Should I use Supervisor :confused:?

When do I use Supervisor?

You use Supervisor when you need to start multiple independent processes in a container.

Let me give you an example, let's say you create a container with Nginx + uWSGI + Django, all in the same container. Then, in this case, Supervisor will be suitable.

Though, if you are using Docker, having Nginx and uWSGI + Django separate is better ( meaning, separate them in two different containers )

Then, use docker-compose to manage the containers; which is this example's method.

Then you will ask, how do I manage containers which unexpectedly exit :confused:?

In this case, you can take a look at the [docker-compose.yml](https://github.com/twtrubiks/docker-django-nginx-uswgi-postgres-tutorial/blob/master/docker-compose.yml) which uses `restart=always` to solve this problem. It will help you restart the container when the container exits unexpectedly :relaxed:

## Conclusion

This also my first time setting up Django + Nginx + uWSGI + Postgres, during which I had to mess around with the configuration for a very long time :scream:. But, I wholeheartedly recommend Docker.

Using Docker was really fun, even when I brake the project, I can just start over again. It's fast and, through this exercise, you will see that Nginx actually has a lot of features which you can play with such as the Load Balancer. Through which you can better understand servers; I myself understood a lot from it.

In short, I recommend that everyone get their hands dirty, follow my footsteps and play with it. I believe that more or less everyone will learn something.

I am also new to Docker, if I have done anything wrong, please let me know, I will make the necessary changes :blush:

If you are still learning to read, read more :satisfied:

* [實戰 Docker + Django + Nginx + uWSGI + Postgres - Load Balance 📝](https://github.com/twtrubiks/docker-django-nginx-uwsgi-postgres-load-balance-tutorial)
* [Docker Swarm 基本教學 - 從無到有 Docker-Swarm-Beginners-Guide📝](https://github.com/twtrubiks/docker-swarm-tutorial)

## Environment

* Mac
* Python 3.8.2
* windows 10
* Liunx

## Reference

* [https://docs.docker.com/](https://docs.docker.com/)
* [uwsgi-docs](https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html)

## Donation

This document is the result of my own internalization after researching. If it has helped you, and you would like to encourage me, you're welcome to buy me a cup of coffee :laughing:

![alt tag](https://i.imgur.com/LRct9xa.png)

[Sponsor Me](https://payment.opay.tw/Broadcaster/Donate/9E47FDEF85ABE383A0F5FC6A218606F8)

## License

MIT license
