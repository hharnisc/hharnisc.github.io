---
layout: post
title: Developing Inside Docker Containers with OS X (2016)
tags:
- Docker
---

Last August, I set out to create a streamlined development environment for building [microservices](http://martinfowler.com/articles/microservices.html). The goal was to leverage [Docker](https://docker.com) to keep things consistent between development and production environments. For the most part I was able to achieve this goal, however the solution involved a bunch of time consuming setup and configuration. Since then, the Docker has released a [private beta](https://beta.docker.com/) that allows you to run Docker containers locally (well, on the slick [xhyve](https://github.com/mist64/xhyve) hypervisor). So as a followup to [Developing Inside Docker Containers With OS X](/2015/09/16/developing-inside-docker-containers-with-osx.html) I'd like to share some of the improvements this has made to the developer experience.

#### The Previous Setup Had Issues

As a refresher here's a list of the original goals I had in mind:

- consistency between development and production environments
- live app reloading when changing code on the host
- one command development environment setup
- logging, monitoring and debugging tools when developing

The big problem with the previous setup was all of the extra configuration necessary to get hot reload working. You had to use Vagrant and Rsync to get past the [Virtualbox Inotify issue](https://www.virtualbox.org/ticket/10660). To top this off, spinning up a VM with Vagrant, installing the docker compose provisioner, installing the vagrant rsync utilities, and pulling all of the images was slow as hell. Virtualbox still hasn't fixed this and it seems likely it will always be broken. But thankfully Docker's beta release fixes this.

#### How Is Docker Beta Different?

##### Docker Original

In the current stable (6/2016), the Docker host runs on a light weight version of linux (boot2docker) which runs inside of a Virtualbox VM. The Docker client communicates with the Docker host running on the boot2docker instance. The Docker team took this implementation so far as to feel pretty close to native development and life was good until you wanted to run tools that required inotify. So you've got to choose between a pile of tools and configuration or developing in linux. Not the worst thing in the world, but we can do better!

<img src="/images/posts/docker-original-layers.png" width="100%">

##### Docker Beta

In the beta, the Docker team has swapped out Virtualbox for the xhyve hypervisor (👏👏👏). xhyve runs Alpine linux, which runs the Docker host. So the Docker client running in OS X communicates with the Docker host on Alpine. This stack has a few key advantages over the Docker Original stack:

- Access containers using *localhost*, rather than the VM IP address
- xhyve runs entirely in user space and has [access to the host file system](https://github.com/mist64/xhyve/blob/793d17ccffa9a1f74f6f1a4997e73cb2e1496296/include/xhyve/firmware/fbsd.h#L28-L52) so tools like `nodemon` and `watchman` behave they same way they do natively
- there's more but I'll leave that for another post

<img src="/images/posts/docker-for-mac-layers.png" width="100%">

#### The New Setup

With Docker beta I was able to remove **ALL** of the extra cruft required to get hot reload working. Vagrant *gone*, rsync *gone*. That is to say I ran `docker-compose up` with a development container, changed some code, and everything *just worked*. I could give the entire Docker team a hug and a handshake, but I won't, because that's weird. Here's an example `docker-compose.yml` file:

{% highlight YAML linenos=table %}
version: '2'

services:
  my-service:
    build: .
    command: npm run dev
    ports:
      - "8080:8080"
    volumes:
      - ./service/src:/service/src
{% endhighlight %}

That's about it, run your docker container in dev mode (for me that was using nodemon to watch for src changes) and mount the host volume. This will work in Docker for Mac, Linux, and would assume Windows.

#### A Yeoman Generator

I've created a [Yeoman](http://yeoman.io/) generator to capture the new simplified workflow:

[https://github.com/hharnisc/generator-service-native-docker](https://github.com/hharnisc/generator-service-native-docker)

As always submit PRs if you find anything that seems odd.

#### Conclusion

Nothing but impressed with the Docker beta release. It did everything I wanted wanted it to. The only odd thing that happened was that Docker beta needed to be restarted *once* after I put my machine to sleep (had to rebuild vagrant boxes 2 times in the same time span). I didn't run into anything I'd consider a rough edge... so I hope the Docker team releases this soon!
