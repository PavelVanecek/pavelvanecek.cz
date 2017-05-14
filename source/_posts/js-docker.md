---
title: Javascript, Docker, and staying sane
date: 2016-10-10 22:22:22
---

_This is a rewrite of a [talk at ngParty III](https://www.meetup.com/ngParty/events/231965205/)._

At [MSD](https://www.msdit.cz), we are writing a lot of Javascript. We run all that using Docker. I would like to show you how these two fit together, why is that a good thing and what we have learned using Docker for almost two years.

## What is Docker?

There are many many articles explaining Docker so I will be brief here.

Docker is a tool to help you package your applications to containers. Think of container as virtual machine, but not quite: containers share kernel and are much faster. It is more like chroot on steroids.
Why do we use Docker? Our environment is reproducible, robust, and idempotent. _“It works on my machine”_ is a solved problem. Setup and deploy is much easier. Our environment is code which means we version it in git and we can do code reviews.

If you want to read more, these are great resources:
[What is Docker? By Docker, Inc.](https://www.docker.com/what-docker)
[A Not Very Short Introduction to Docker by Anders Janmyr](https://www.jayway.com/2015/03/21/a-not-very-short-introduction-to-docker/)
[Getting Started with Docker for the Node.js Developer by Heitor Tashiro Sergent
](https://www.airpair.com/node.js/posts/getting-started-with-docker-for-the-nodejs-dev)
[Docker Overview by Docker, Inc.](https://docs.docker.com/engine/understanding-docker/)
[Dockerizing a Node.js web app by Node.js Foundation](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)

There are a few core concepts of Docker:
- Containers
- Images
- Dockerfile
- Registry

### Containers

![Not exactly a Docker container but almost](/images/js-docker/container.jpg)
<center> _Not exactly a Docker container but almost_ </center>

Container is a shell for your application. Docker uses a metaphor with physical containers: the kind you put on ships and move them around. The metaphor works very well. The shipping company does not need to bother how to handle particular packages and does not need equipment to move them. Instead you package stuff into a standardized container.
Same thing with Docker containers: they have defined behavior and API so that you do not really care what is inside it.
Containers also help you isolate environments in a way similar to virtual machines, but much faster and with smaller footprint. VM can start in seconds, or even minutes? Spinning up a container comes with delay of several milliseconds.

### Images

How does one run a container? Using Docker image.

![Docker images do not actually look like Mona Lisa](/images/js-docker/mona-lisa.jpg)
<center> _Docker images do not actually look like Mona Lisa_ </center>

Docker image is a snapshot, which works as container filesystem. Image on its own is static, does not run anything and does not have any intermediate state. It only allows creating containers.

![Images have layers, just like onions. Or Tiramisu](/images/js-docker/layers.jpg)
<center> _Images have layers, just like onions. Or Tiramisu_ </center>

Images come in layers. Each layer is an append-only diff. Images can extend another images: it just adds a new layers on top of the other one.
Images can share layers. If you have 10 images all based off base Node.js image, the common part is only stored once.
So, deleting a file does not make the image smaller! It is still there in the previous layer.

### Registry

![Docker registry might look like this. Your mileage may vary](/images/js-docker/dox.jpg)
<center> _Docker registry might look like this. Your mileage may vary_ </center>

Docker registry is an image store. You may use [the public one](https://hub.docker.com/explore/), pay for [private one](https://docs.docker.com/docker-hub/repos/) managed by Docker Inc. or manage your own private repository.

### Docker CLI

Docker is a [command line tool](https://docs.docker.com/engine/reference/commandline/). There are the most common commands you will use every day:

![Most common Docker CLI commands](/images/js-docker/cli.png)

- `docker run` runs new container from given image
- `docker build` uses Dockerfile to build new image
- `docker push` and `docker pull` help you interact with a registry

### Dockerfile

```Dockerfile
FROM node:6
MAINTAINER Pavel Vanecek <email@pavelvanecek.cz>

WORKDIR /app
COPY ./package.json /tmp/package.json
RUN cd /tmp &&\
    npm install &&\
    cp -a /tmp/node_modules /app/ &&\
    rm /tmp/package.json

COPY . .
RUN npm test
```
<center> _A sample Dockerfile_ </center>

[Dockerfile is a plain text file](https://docs.docker.com/engine/reference/builder/) with instructions how to build an image. Each instruction creates new layer.

There are some important bits you should not miss:

```Dockerfile
FROM node:6
```
_Always_ specify base image version. The default is `:latest` and using latest will break your builds eventually. We tried.


```Dockerfile
COPY ./package.json /tmp/package.json
RUN cd /tmp &&\
    npm install &&\
    cp -a /tmp/node_modules /app/ &&\
    rm /tmp/package.json
```
Docker images come in layers. Remember? Docker caches layers by default. Cache speeds up build, but it can bite you too.
How is cache invalidated? For almost all commands, changing the instruction discards cached layer. For `COPY` command, Docker compares file hash and discards cache every time its content changed. This means that if you do a naive installation, Docker will install all packages over and over again. That can take 10 minutes or more. You do not want that.

```Dockerfile
# Copy current folder from host fs to image.
# Any change in source code will invalidate installed packages!
COPY . /app
# Runs more often that you would like
RUN npm install
```
<center> _Please do not do this at home_ </center>

Once you copy `package.json` first, Docker caches all packages regardless of your source code changes. The cache invalidates once you change contents of package.json: which is what you want it to do anyway.

This practice is not optimal; [some suggest](http://jdlm.info/articles/2016/03/06/lessons-building-node-app-docker.html) using a volume, or a [npm shrinkwrap file](https://docs.npmjs.com/cli/shrinkwrap).
We do neither, on purpose.
We value that running a full rebuild from time to time does help us identify some issues we would have not found otherwise.
It makes some builds last longer, and build breaks more often. But it breaks sooner, and we are aware of the bug much earlier than we would have been otherwise. I believe that is worth the tradeoff.

```Dockerfile
COPY . .
RUN npm test
```

Every built image comes with passing tests. This helps us make sure that we ship tested code. Every change in source code invalidates the layer and makes tests run again.

There are several pieces missing in this article which you may need:
- [Volumes](https://docs.docker.com/engine/tutorials/dockervolumes/)
- [Ports](https://www.ctl.io/developers/blog/post/docker-networking-rules/)
- [Entrypoint and / or CMD](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#/cmd)
- [Labels](https://docs.docker.com/engine/userguide/labels-custom-metadata/)
- [Configuring user other than root](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#/user)
- [apt-getting more packages](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#/build-cache)

### .dockerignore

```
node_modules/
```

Your node_modules folder probably contains native packages compiled for different platform and incompatible with Linux containers. You need the image to build its own.
As a bonus, it speeds up build a lot: Docker for Mac suffers from slow filesystem operations. Ignoring unused node_modules helps.

See the [.dockerignore reference](https://docs.docker.com/engine/reference/builder/#dockerignore-file)


### Environment variables

```javascript
const OUTPUT_FILENAME = process.env.OUTPUT_FILENAME || './output.txt'
const PORT = process.env.PORT || 8080
```

We abandoned files and configure our applications using shell environment variables. With Docker toolset, it is easier to manage the environment than figure our which file belongs where.

### Logging

All logging is best done to standard output: `process.stdout`. The tooling expects that so do not bother with your application being fancy with log files.

## Docker Compose

Once you follow the advice above you will have a Node.js application running inside a container. But the app usually does not run alone; it requires another services. Do they run in containers? Of course!

Now running one container is simple. Running several of requires some care. This is where Docker Compose comes to rescue.

![Managing containers can be easy](/images/js-docker/Container_ship_Hanjin_Taipei.jpg)
<center> _Managing containers can be easy_ </center>

Docker compose is great tool that helps you manage orchestrating multiple containers together.
It is useful even when you have a single container: remembering all command line flags is daunting.

```YAML
version: '2'

services:

  mysql:
    image: mysql:5.7
    restart: always
    volumes:
      - "./.data/db:/var/lib/mysql"
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: wordpress

  web:
    links:
      - mysql
    build:
      context: .
      dockerfile: run.Dockerfile
    ports:
      - 80:80
    restart: always
```
<center> _Sample docker-compose.yml file_ </center>

This is how a typical `docker-compose.yml` looks like. It spins a _web_ service providing HTML pages, and _mysql_ service holding the data.
Let us go through some interesting pieces:

```YAML
    image: mysql:5.7
    # ... or
    build:
      context: .
```

Docker will download existing images from registry automatically. You may also build your own when launching the stack.

```YAML
    volumes:
      - "./.data/db:/var/lib/mysql"
```
Volumes allow persisting files from the container on host filesystem; here the mysql data.

```YAML
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: wordpress
```

Remember the environment variables? It comes in handy.
Do not include production database passwords in a compose file! Consider this an example.
Handling credentials is a talk on its own; we usually use tools like Vault, or get the passwords from somewhere completely different place. Usually, we encrypt the important pieces using [Ansible Vault](http://docs.ansible.com/ansible/playbooks_vault.html) and generate docker compose files using Ansible.
You most certainly do not want to have your [AWS credentials exposed](http://www.theregister.co.uk/2015/01/06/dev_blunder_shows_github_crawling_with_keyslurping_bots/).

```YAML
    links:
      - mysql
```
Containers by default do not see each other. You may link them together using a [private network](https://docs.docker.com/compose/networking/) created by Docker.
The great thing is that we usually hardcode hostnames in the app and configure the environment instead:

```javascript
const DB_HOSTNAME = 'mysql'
```

With Docker links, this comes for free. When running the application directly on hostname, one can edit their `/etc/hosts` file for the same effect (this is what Docker does with links anyway).

There is a caveat though. Docker knows to boot containers in correct order, but it is not smart enough to [wait until the process inside is ready](https://github.com/docker/compose/issues/374). That may take seconds or minutes. Your application must count for this and retry failed connections to database before giving up.

![This thread has some great advice how to solve the linked container problem](/images/js-docker/docker-compose-delayed-start.png)
<center> _This thread has some great advice how to solve the linked container problem_ </center>

```YAML
    ports:
      - 80:80
```

By default, container ports are [not exposed](https://docs.docker.com/engine/userguide/networking/) to the outside world. It is not usually necessary; you do not want your MySQL port to be accessible from anywhere else but the application.
Unless you actually want to expose some ports. Docker lets you map easily.

```YAML
    restart: always
```
Thanks to [built-in Docker restart policies](https://docs.docker.com/engine/admin/host_integration/), we no longer use any other process managers. We have used and abandoned init.d scripts, foreman.js and pm2. Docker can keep your containers alive.

## What good came out of it

![Not a very good commit](/images/js-docker/package-json-bug.png)
<center> _Not a very good commit_ </center>

This is an example of a bad commit, which would break the application in a subtle way. A developer deleted some dependencies by mistake. Tests are passing, application works on staging and production! All because the build script did not have `npm prune` in it so that the packages were not actually removed. This would only surface as soon as new developer joins the team, or a new server is provisioned. Potentially few weeks or months after this commit.
Remember the full builds? This would have failed the build.

By the way: We found this bug during code review. Do you do code reviews? You should.

![Our README got much simpler](/images/js-docker/simpler-readme.png)
<center> _Our README got much simpler_ </center>

This is an example of WordPress README instructions. At first, there were no installation instructions. Bad, mind-twisting process of installing everything took several hours. Trial and error driven deployment.
On the left is a screenshot of README after we introduced Docker. It alone made the process predictable and faster: it only took two hours.
On the right is README instructions after we added a `docker-compose.yml` file. Now, a new developer can get from zero to full productivity in five minutes.

Using Docker made our build and deployment more robust, reliable, deterministic, and faster.

On the other hand, local development on OSX is still faster in some platforms. Node.js and webpack requires file watching during development; Docker used to have issues. I myself usually default to [nvm](https://github.com/creationix/nvm).
Compiling and running Scala applications requires copying several GB of libraries. That is [heartbreakingly slow](https://forums.docker.com/t/file-access-in-mounted-volumes-extremely-slow-cpu-bound/8076) on Mac's xhyve.

But it is definitely possible. Check out the talk by David Blurton for some great insight:
<center>
  <iframe width="560" height="315" src="https://www.youtube.com/embed/zcSbOl8DYXM" frameborder="0" allowfullscreen></iframe>
</center>

## That's it

In this article, I have just scratched the surface of what is possible with Docker. The Docker world is vast, is changing constantly and I believe in bright future of containers.

## Update (Nov 7 2016)

I found several great articles that pretty much match our experience:

- [Docker in Production: A History of Failure](https://thehftguy.wordpress.com/2016/11/01/docker-in-production-an-history-of-failure/) by _The HFT Guy_
- [Docker in Production: A retort](https://patrobinson.github.io/2016/11/05/docker-in-production/) by _Sysadmin 4 lyfe_
- [Why Docker is Not Yet Succeeding Widely in Production](http://sirupsen.com/production-docker/) by _Simon Hørup_ Eskildsen_
- [Docker Not Ready for Prime Time](http://blog.goodstuff.im/docker_not_prime_time) by _David P. Pollak_
- [Thou shalt not run a database inside a container](https://patrobinson.github.io/2016/11/07/thou-shalt-not-run-a-database-inside-a-container/) by _Sysadmin 4 lyfe_
