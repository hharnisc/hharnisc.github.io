---
layout: post
title: Developing Inside Docker Containers with OS X
tags:
- Docker
- Vagrant
---

One of the biggest selling points of [Docker](https://www.docker.com/) is consistency between your development and production environments. But there are plenty of challenges when setting up a development environment inside of a Docker image (especially on an OS X host). After spending some time getting familiar with Docker and Vagrant, I learned a number of things I felt would be valuable to share.

For my efforts, I was able to achieve a development environment with the following attributes:

- consistency between development and production environments
- live app reloading when changing code on the host
- one command development environment setup
- logging, monitoring and debugging tools when developing

#### Day 1: Docker Is Great!

Even the Docker haters have this moment when first using Docker: 😻 You find yourself thinking _I'm going to **Dockerize** Everything_, so you add a `Dockerfile` to your Node app and build it.

**Dockerfile**

{% highlight Dockerfile linenos=table %}
FROM mhart/alpine-node
WORKDIR /app
COPY app/ .
EXPOSE 8080
CMD ["npm", "start", "--", "--port", "8080"]
{% endhighlight %}


Now your deployment pipeline gets a little simpler since you can use one of a few services that can run containers and throw your chef scripts under a bus. But then **it** happens, you deploy a container to production and it **doesn't work**.

You wonder what could've gone wrong. You've been using Docker to deployments so it should have just worked. But you've been developing and testing on your local machine in a completely different environment. You're basically just using Docker to package your app.

#### Let's Build Stuff Inside Docker Containers

Now you're a little smarter and use [Docker Compose](https://docs.docker.com/compose/) to mount a volume from your host machine inside your container.

**Dockerfile**

{% highlight Dockerfile linenos=table %}
FROM mhart/alpine-node
WORKDIR /app
COPY app/ .
{% endhighlight %}

**docker-compose.yml**

{% highlight YAML linenos=table %}
web:
  build: .
  command: "npm start"
  ports:
    - "8080:8080"
  volumes:
    - ./app:/app
{% endhighlight %}

You can easily build and run docker containers with Docker Compose

{% highlight bash linenos=table %}
$ docker-compose build ; docker-compose up
{% endhighlight %}

With this change the mounted folder on the Docker container is kept in sync with the host. You'd expect that you should be able to use one of the continuous build tools for node like [nodemon](http://nodemon.io/) to streamline the development process. So you modify your docker-compose file

**docker-compose.yml**

{% highlight YAML linenos=table %}
web:
  build: .
  command: sh -c 'npm install -g nodemon ; nodemon index.js'
  ports:
    - "8080:8080"
  volumes:
    - ./app:/app
{% endhighlight %}

The app now starts up with `nodemon` but it doesn't restart when you change code. SHIT.

Now this is where things got hairy and I imagine many fell off the cliff. But I can assure you there are good things ahead, great things even. I spent a couple nights irritable and agitated trying several different approaches so (hopefully) you don't have to.

#### I-notify You When I Feel Like It

So nodemon (and many other file watching tools) use [Inotify](https://en.wikipedia.org/wiki/Inotify) to detect file changes. They do this because it's fast. It's part of the Linux kernel subsystem and publishes changes to subscribed applications rather than having to scan the files for changes.

After some StackOverflowing, Googling, and Github Issuing I stumbled across this [won't fix](https://www.virtualbox.org/ticket/10660) bug for Virtualbox. I translates to **shared folders can't trigger Inotify events**. DOUBLE SHIT. This means there's a whole class of syncing tools that simply won't work.

#### There's A Light

Just because virtualbox shared folders can't generate Inotify events doesn't mean that Inotify events aren't fired within the container. If you change a file on the container with `vim` this does cause nodemon to trigger a restart. So it's possible that using a syncing tool could solve this problem. I evaluated the following tools:

- **[Docker Unison](https://github.com/leighmcculloch/docker-unison)** - Docker + [Unison](http://www.cis.upenn.edu/~bcpierce/unison/) It worked but was buggy
- **[Hodor](https://github.com/gansbrest/hodor)** - Uses unison under the hood, was not able to get this to work and added another layer of complexity with _YET ANOTHER_ configuration syntax
- **[Docker OSX Dev](https://github.com/brikis98/docker-osx-dev)** - Worked about 90% of the time, and I really wanted this to work. But it kept crashing and would fail silently.

#### Enter Vagrant

[Vagrant](https://www.vagrantup.com/) has been around since 2012 ([as far as I can tell](http://www.linuxjournal.com/content/introducing-vagrant)). So it's had some time to mature. Vagrant focuses on creating and configuring virtual environments. It can provision a number of environments and has a plugin system with an active community. With previous projects, I found Vagrant added more complexity than needed. However it does solve some problems with client/host syncing that aren't easy or simple.  I spent some time working through the examples and digging through documentation until I had the tools needed. With Vagrant syncing and a couple of plugins I was able to achieve the dream of typing one command and getting a reproducible development environment.

<img src="/images/posts/vagrant-layers.png" width="80%">

Vagrant makes it very easy to employ [Rsync](https://en.wikipedia.org/wiki/Rsync) as a mechanism to sync the host onto the client (yes, one sync way but more on that later). Using Rsync on the Virtualbox instance get's us around the **Inotify problem**. When the Rsync client on the VM get's a change from the host, the Rsync client modifies the file from within the client triggering Inotify. So our continuous build system works as expected! And Rsync is [really really fast](https://gist.github.com/KartikTalwar/4393116), so it can handle just about anything you can throw at it.

_But there's still one more thing_. The auto-sync feature in Vagrant doesn't work very well for me. It's responsible for listening on the host and triggering the sync to the Vagrant managed VM. Thankfully, the Vagrant plugin community picked up the slack here with a nice little tool called [vagrant-gatling-rsync](https://github.com/smerrill/vagrant-gatling-rsync) and [vagrant-docker-compose](https://github.com/leighmcculloch/vagrant-docker-compose).

Here's what my Vagrantfile, docker-compose.yml and (optional) bash script that let's you start this up with one command.

**Vagrantfile**

{% highlight Ruby linenos=table %}
Vagrant.configure("2") do |config|
  # feed the monster
  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
    v.cpus = 2
  end

  # using ubuntu host
  config.vm.box = "ubuntu/trusty64"

  # use rsync to keep host in sync with guest VM
  config.vm.synced_folder ".", "/sync", type: "rsync",
    rsync__exclude: ".git/",
    rsync__exclude: "node_modules/"

  # provision with docker and docker-compose
  config.vm.provision :docker
  # when running docker compose always rebuild and run
  config.vm.provision :docker_compose,
    yml: "/sync/docker-compose.yml",
    rebuild: true,
    run: "always"

  # don't automatically start syncing,
  # we'll kick that off in the startup script
  config.gatling.rsync_on_startup = false

  # forward ports
  config.vm.network :forwarded_port, guest: 8080, host: 8080 # service
end

{% endhighlight %}

**Dockerfile**

{% highlight Dockerfile linenos=table %}
FROM mhart/alpine-node
WORKDIR /app
COPY app/ .
{% endhighlight %}

**docker-compose.yml**

{% highlight YAML linenos=table %}
app:
  build: .
  command: sh -c 'npm install --unsafe-perm ; npm install -g gulp ; gulp dev'
  ports:
    - "8080:8080"
  volumes:
    - /sync/service:/service #/sync is defined in the vagrant file
{% endhighlight %}

**start-dev.sh**

{% highlight Bash linenos=table %}
#!/bin/bash
# start vagrant and the vagrant sync tool
vagrant up
echo "⚡⚡ TIME TO BUILD COOL STUFF ⚡⚡"
vagrant gatling-rsync-auto
{% endhighlight %}

#### A Yeoman Generator

I've created a [Yeoman](http://yeoman.io/) generator to capture this workflow.

[https://github.com/hharnisc/generate-service](https://github.com/hharnisc/generate-service)

It generates a simple **Node microservice** (with hot reloading in the container), a central logging (**ELK stack**) service and uses **RabbitMQ** to communicate between services. Please use this for informational purposes or as a starting point for your project. It's not quite ready for production. I'm planning on publishing this to NPM when I get some more feedback, I love PRs!

#### Conclusion

There are a few takeaways from this experience:

- It's astounding how one **[won't fix](https://www.virtualbox.org/ticket/10660)** bug in Virtualbox has opened the door for dozens of mostly broken attempts to fix it.
- If you're using Docker containers and want the best experience _right now_ you might want to be living on one of the popular Linux distros.
- After going through walkthroughs, blog posts, and documentation it's clear that there are 2 distinct groups using Docker containers, **Software Developers** and **Ops/Infra** People.

Docker has given language to Software Developers and Ops/Infra types that enables clearer communication. As time goes on I think the line will blur between these two disciplines because of tools like Docker. *And this is a good thing* because these are problems worth solving. It seems like someone new comes along every day and fills in one more piece of the pipeline -- now we just need to connect it all.

UPDATE: Check out my follow up post [Developing Inside Docker Containers with OS X (2016)](/2016/06/16/developing-inside-docker-containers-with-osx-2016.html) and the new [Yeoman generator](https://github.com/hharnisc/generator-service-native-docker). Much has been changed in the Docker beta release!
