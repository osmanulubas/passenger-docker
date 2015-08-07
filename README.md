# Docker base images for Ruby, Python, Node.js and Meteor web apps

<center><img src="http://blog.phusion.nl/wp-content/uploads/2012/07/Passenger_chair_256x256.jpg" width="196" height="196" alt="Phusion Passenger"> <img src="http://blog.phusion.nl/wp-content/uploads/2013/11/docker.png" width="233" height="196" alt="Docker"></center>

Passenger-docker is a set of [Docker](https://www.docker.com) images meant to serve as good bases for **Ruby, Python, Node.js and Meteor** web app images. In line with [Phusion Passenger](https://www.phusionpassenger.com/)'s goal, passenger-docker's goal is to make Docker image building for web apps much easier and faster.

Why is this image called "passenger"? It's to represent the ease: you just have to sit back and watch most of the heavy lifting being done for you. Passenger-docker is part of a larger and more ambitious project: to make web app deployment ridiculously simple, to heights never achieved before.

**Relevant links:**
 [Github](https://github.com/phusion/passenger-docker) |
 [Docker registry](https://registry.hub.docker.com/u/phusion/passenger-full/) |
 [Discussion forum](https://groups.google.com/d/forum/passenger-docker) |
 [Twitter](https://twitter.com/phusion_nl) |
 [Blog](http://blog.phusion.nl/)

---------------------------------------

**Table of contents**

 * [Why use passenger-docker?](#why_use)
 * [About passenger-docker](#about)
   * [What's included?](#whats_included)
   * [Memory efficiency](#memory_efficiency)
   * [Image variants](#image_variants)
 * [Inspecting the image](#inspecting_the_image)
 * [Using the image as base](#using)
   * [Getting started](#getting_started)
   * [The `app` user](#app_user)
   * [Using Nginx and Passenger](#nginx_passenger)
     * [Adding your web app to the image](#adding_web_app)
     * [Configuring Nginx](#configuring_nginx)
     * [Setting environment variables in Nginx](#nginx_env_vars)
     * [Application environment name (`RAILS_ENV`, `NODE_ENV`, etc)](#app_env_name)
   * [Using Redis](#redis)
   * [Using memcached](#memcached)
   * [Additional daemons](#additional_daemons)
   * [Selecting a default Ruby version](#selecting_default_ruby)
   * [Running scripts during container startup](#running_startup_scripts)
   * [Upgrading the operating system inside the container](#upgrading_os)
   * [Upgrading Passenger to the latest version](#upgrading_passenger)
 * [Container administration](#container_administration)
   * [Running a one-shot command in a new container](#oneshot)
   * [Running a command in an existing, running container](#run_inside_existing_container)
   * [Login to the container via `docker exec`](#login_docker_exec)
     * [Usage](#docker_exec)
   * [Login to the container via SSH](#login_ssh)
     * [Enabling SSH](#enabling_ssh)
     * [About SSH keys](#ssh_keys)
     * [Using the insecure key for one container only](#using_the_insecure_key_for_one_container_only)
     * [Enabling the insecure key permanently](#enabling_the_insecure_key_permanently)
     * [Using your own key](#using_your_own_key)
     * [The `docker-ssh` tool](#docker_ssh)
   * [Inspecting the status of your web app](#inspecting_web_app_status)
   * [Logs](#logs)
 * [Switching to Phusion Passenger Enterprise](#enterprise)
 * [Building the image yourself](#building)
 * [Conclusion](#conclusion)

---------------------------------------

<a name="why_use"></a>
## Why use passenger-docker?

Why use passenger-docker instead of doing everything yourself in Dockerfile?

 * Your Dockerfile can be smaller.
 * It reduces the time needed to write a correct Dockerfile. You won't have to worry about the base system and the stack, you can focus on just your app.
 * It sets up the base system **correctly**. It's very easy to get the base system wrong, but this image does everything correctly. [Learn more.](https://github.com/phusion/baseimage-docker#contents)
 * It drastically reduces the time needed to run `docker build`, allowing you to iterate your Dockerfile more quickly.
 * It reduces download time during redeploys. Docker only needs to download the base image once: during the first deploy. On every subsequent deploys, only the changes you make on top of the base image are downloaded.

<a name="about"></a>
## About the image

<a name="whats_included"></a>
### What's included?

*Passenger-docker is built on top of a solid base: [baseimage-docker](http://phusion.github.io/baseimage-docker/).*

Basics (learn more at [baseimage-docker](http://phusion.github.io/baseimage-docker/)):

 * Ubuntu 14.04 LTS as base system.
 * A **correct** init process ([learn more](http://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/)).
 * Fixes APT incompatibilities with Docker.
 * syslog-ng.
 * The cron daemon.
 * [Runit](http://smarden.org/runit/) for service supervision and management.

Language support:

 * Ruby 1.9.3, 2.0.0, 2.1.6, and 2.2.2; JRuby 9.0.0.0.
   * 2.2.2 is configured as the default.
   * MRI Ruby is installed through [the Brightbox APT repository](https://launchpad.net/~brightbox/+archive/ruby-ng). We're not using RVM!
   * JRuby is installed from source, but we register an APT entry for it.
   * JRuby uses OpenJDK 8 from [the openjdk-r PPA](https://launchpad.net/~openjdk-r/+archive/ubuntu/ppa).
 * Python 2.7 and Python 3.4.
 * Node.js 0.12, through [NodeSource's APT repository](https://nodesource.com/).
 * A build system, git, and development headers for many popular libraries, so that the most popular Ruby, Python and Node.js native extensions can be compiled without problems.

Web server and application server:

 * Nginx 1.6. Disabled by default.
 * [Phusion Passenger 4](https://www.phusionpassenger.com/). Disabled by default (because it starts along with Nginx).
   * This is a fast and lightweight tool for simplifying web application integration into Nginx.
   * It adds many production-grade features, such as process monitoring, administration and status inspection.
   * It replaces (G)Unicorn, Thin, Puma, uWSGI.
   * Node.js users: [watch this 4 minute intro video](http://vimeo.com/phusionnl/review/84945384/73fe7432ee) to learn why it's cool and useful.

Auxiliary services and tools:

 * Redis 2.6, through [Rowan's Redis PPA](https://launchpad.net/~rwky/+archive/redis). Disabled by default.
 * Memcached. Disabled by default.

<a name="memory_efficiency"></a>
### Memory efficiency

Passenger-docker is very lightweight on memory. In its default configuration, it only uses 10 MB of memory (the memory consumed by bash, runit, syslog-ng, etc).

<a name="image_variants"></a>
### Image variants

Passenger-docker consists of several images, each one tailor made for a specific user group.

**Ruby images**

 * `phusion/passenger-ruby19` - Ruby 1.9.
 * `phusion/passenger-ruby20` - Ruby 2.0.
 * `phusion/passenger-ruby21` - Ruby 2.1.
 * `phusion/passenger-ruby22` - Ruby 2.2.
 * `phusion/passenger-jruby90` - JRuby 9.0.0.0.

**Node.js and Meteor images**

 * `phusion/passenger-nodejs` - Node.js 0.11.

**Other images**

 * `phusion/passenger-full` - Contains everything in the above images. Ruby, Python, Node.js, all in a single image for your convenience.
 * `phusion/passenger-customizable` - Contains only the base system, as described in ["What's included?"](#whats_included). Ruby, Python and Node.js are not preinstalled. This image is meant to be further customized through your Dockerfile. For example, using this image you can create a custom image that contains only Ruby 2.0, Ruby 2.1 and Node.js.

In the rest of this document we're going to assume that the reader will be using `phusion/passenger-full`, unless otherwise stated. Simply substitute the name if you wish to use another image.

<a name="inspecting_the_image"></a>
## Inspecting the image

To look around in the image, run:

    docker run --rm -t -i phusion/passenger-full bash -l

You don't have to download anything manually. The above command will automatically pull the passenger-docker image from the Docker registry.

<a name="using"></a>
## Using the image as base

<a name="getting_started"></a>
### Getting started

There are several images, e.g. `phusion/passenger-ruby21` and `phusion/passenger-nodejs`. Choose the one you want. See [Image variants](#image_variants).

So put the following in your Dockerfile:

    # Use phusion/passenger-full as base image. To make your builds reproducible, make
    # sure you lock down to a specific version, not to `latest`!
    # See https://github.com/phusion/passenger-docker/blob/master/Changelog.md for
    # a list of version numbers.
    FROM phusion/passenger-full:<VERSION>
    # Or, instead of the 'full' variant, use one of these:
    #FROM phusion/passenger-ruby19:<VERSION>
    #FROM phusion/passenger-ruby20:<VERSION>
    #FROM phusion/passenger-ruby21:<VERSION>
    #FROM phusion/passenger-ruby22:<VERSION>
    #FROM phusion/passenger-jruby90:<VERSION>
    #FROM phusion/passenger-nodejs:<VERSION>
    #FROM phusion/passenger-customizable:<VERSION>

    # Set correct environment variables.
    ENV HOME /root

    # Use baseimage-docker's init process.
    CMD ["/sbin/my_init"]

    # If you're using the 'customizable' variant, you need to explicitly opt-in
    # for features. Uncomment the features you want:
    #
    #   Build system and git.
    #RUN /pd_build/utilities.sh
    #   Ruby support.
    #RUN /pd_build/ruby1.9.sh
    #RUN /pd_build/ruby2.0.sh
    #RUN /pd_build/ruby2.1.sh
    #RUN /pd_build/ruby2.2.sh
    #RUN /pd_build/jruby9.0.sh
    #   Python support.
    #RUN /pd_build/python.sh
    #   Node.js and Meteor support.
    #RUN /pd_build/nodejs.sh

    # ...put your own build instructions here...

    # Clean up APT when done.
    RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

<a name="app_user"></a>
### The `app` user

The image has an `app` user with UID 9999 and home directory `/home/app`. Your application is supposed to run as this user. Even though Docker itself provides some isolation from the host OS, running applications without root privileges is good security practice.

Your application should be placed inside /home/app.

<a name="nginx_passenger"></a>
### Using Nginx and Passenger

Before using Passenger, you should familiarise yourself with it by [reading its documentation](https://www.phusionpassenger.com).

Nginx and Passenger are disabled by default. Enable them like so:

    RUN rm -f /etc/service/nginx/down

<a name="adding_web_app"></a>
#### Adding your web app to the image

Passenger works like a `mod_ruby`, `mod_nodejs`, etc. It changes Nginx into an application server and runs your app from Nginx. So to get your web app up and running, you just have to add a virtual host entry to Nginx which describes where you app is, and Passenger will take care of the rest.

You can add a virtual host entry (`server` block) by placing a .conf file in the directory `/etc/nginx/sites-enabled`. For example:

    # /etc/nginx/sites-enabled/webapp.conf:
    server {
        listen 80;
        server_name www.webapp.com;
        root /home/app/webapp/public;

        # The following deploys your Ruby/Python/Node.js/Meteor app on Passenger.

        # Not familiar with Passenger, and used (G)Unicorn/Thin/Puma/pure Node before?
        # Yes, this is all you need to deploy on Passenger! All the reverse proxying,
        # socket setup, process management, etc are all taken care automatically for
        # you! Learn more at https://www.phusionpassenger.com/.
        passenger_enabled on;
        passenger_user app;

        # If this is a Ruby app, specify a Ruby version:
        passenger_ruby /usr/bin/ruby2.1;
        # For Ruby 2.0
        passenger_ruby /usr/bin/ruby2.0;
        # For Ruby 1.9.3 (you can ignore the "1.9.1" suffix)
        #passenger_ruby /usr/bin/ruby1.9.1;
    }

    # Dockerfile:
    RUN rm /etc/nginx/sites-enabled/default
    ADD webapp.conf /etc/nginx/sites-enabled/webapp.conf
    RUN mkdir /home/app/webapp
    RUN ...commands to place your web app in /home/app/webapp...

<a name="configuring_nginx"></a>
#### Configuring Nginx

The best way to configure Nginx is by adding .conf files to `/etc/nginx/main.d` and `/etc/nginx/conf.d`. Files in `main.d` are included into the Nginx configuration's main context, while files in `conf.d` are included in the Nginx configuration's http context.

For example:

    # /etc/nginx/main.d/secret_key.conf:
    env SECRET_KEY=123456;

    # /etc/nginx/conf.d/gzip_max.conf:
    gzip_comp_level 9;

    # Dockerfile:
    ADD secret_key.conf /etc/nginx/main.d/secret_key.conf
    ADD gzip_max.conf /etc/nginx/conf.d/gzip_max.conf

<a name="nginx_env_vars"></a>
#### Setting environment variables in Nginx

By default Nginx [clears all environment variables](http://nginx.org/en/docs/ngx_core_module.html#env) (except `TZ`) for its child processes (Passenger being one of them). That's why any environment variables you set with `docker run -e`, Docker linking and `/etc/container_environment`, won't reach Nginx.

To preserve these variables, place an Nginx config file ending with `*.conf` in the directory `/etc/nginx/main.d`, in which you tell Nginx to preserve these variables. For example when linking a PostgreSQL container or MongoDB container:

    # /etc/nginx/main.d/postgres-env.conf:
    env POSTGRES_PORT_5432_TCP_ADDR;
    env POSTGRES_PORT_5432_TCP_PORT;

    # Dockerfile:
    ADD postgres-env.conf /etc/nginx/main.d/postgres-env.conf

By default, passenger-docker already contains a config file `/etc/nginx/main.d/default.conf` which preserves the `PATH` environment variable.

<a name="app_env_name"></a>
#### Application environment name (`RAILS_ENV`, `NODE_ENV`, etc)

Some web frameworks adjust their behavior according to the value some environment variables. For example, Rails respects `RAILS_ENV` while Connect.js respects `NODE_ENV`. By default, Phusion Passenger sets all of the following environment variables to the value **production**:

 * `RAILS_ENV`
 * `RACK_ENV`
 * `WSGI_ENV`
 * `NODE_ENV`
 * `PASSENGER_APP_ENV`

Setting these environment variables yourself (e.g. using `docker run -e RAILS_ENV=...`) will not have any effect, because Phusion Passenger overrides all of these environment variables. The only exception is `PASSENGER_APP_ENV` (see below).

With passenger-docker, there are two ways to set the aforementioned environment variables. The first is through the [`passenger_app_env`](https://www.phusionpassenger.com/library/config/nginx/reference/#passenger_app_env) config option in Nginx. For example:

    # /etc/nginx/sites-enabled/webapp.conf:
    server {
        ...
        # Ensures that RAILS_ENV, NODE_ENV, etc are set to "staging"
        # when your application is started.
        passenger_app_env staging;
    }

The second way is by setting the `PASSENGER_APP_ENV` environment variable from `docker run`

    docker run -e PASSENGER_APP_ENV=staging YOUR_IMAGE

This works because passenger-docker autogenerates an Nginx configuration file (`/etc/nginx/conf.d/00_app_env.conf`) during container boot. This file sets the `passenger_app_env` option in the `http` context. This means that if you already set `passenger_app_env` in the `server` context, running `docker run -e PASSENGER_APP_ENV=...` won't have any effect!

If you want to set a default value while still allowing that to be overridden by `docker run -e PASSENGER_APP_ENV=`, then instead of specifying `passenger_app_env` in your Nginx config file, you should create a `/etc/nginx/conf.d/00_app_env.conf`. This file will be overwritten if the user runs `docker run -e PASSENGER_APP_ENV=...`.

    # /etc/nginx/conf.d/00_app_env.conf
    # File will be overwritten if user runs the container with `-e PASSENGER_APP_ENV=...`!
    passenger_app_env staging;

<a name="redis"></a>
### Using Redis

**Redis is only available in the passenger-customizable and passenger-full images!**

Install and enable Redis:

    # Opt-in for Redis if you're using the 'customizable' image.
    #RUN /pd_build/redis.sh

    # Enable the Redis service.
    RUN rm -f /etc/service/redis/down

The configuration file is in /etc/redis/redis.conf. Modify it as you see fit, but make sure `daemonize no` is set.

<a name="memcached"></a>
### Using memcached

**Memcached is only available in the passenger-customizable and passenger-full images!**

Install and enable memcached:

    # Opt-in for Memcached if you're using the 'customizable' image.
    #RUN /pd_build/memcached.sh

    # Enable the memcached service.
    RUN rm -f /etc/service/memcached/down

The configuration file is in /etc/memcached.conf. Note that it does not follow the Debian/Ubuntu format, but our own, in order to make it work well with runit. The default contents are:

    # These arguments are passed to the memcached daemon.
    MEMCACHED_OPTS="-l 127.0.0.1"

<a name="additional_daemons"></a>
### Additional daemons

You can add additional daemons to the image by creating runit entries. You only have to write a small shell script which runs your daemon, and runit will keep it up and running for you, restarting it when it crashes, etc.

The shell script must be called `run`, must be executable, and is to be placed in the directory `/etc/service/<NAME>`.

Here's an example showing you how to a memached server runit entry can be made.

    ### In memcached.sh (make sure this file is chmod +x):
    #!/bin/sh
    # `setuser` is part of baseimage-docker. `setuser mecached xxx...` runs the given command
    # (`xxx...`) as the user `memcache`. If you omit this, the command will be run as root.
    exec /sbin/setuser memcache /usr/bin/memcached >>/var/log/memcached.log 2>&1

    ### In Dockerfile:
    RUN mkdir /etc/service/memcached
    ADD memcached.sh /etc/service/memcached/run

Note that the shell script must run the daemon **without letting it daemonize/fork it**. Usually, daemons provide a command line flag or a config file option for that.

**Tip**: If you're thinking about running your web app, consider deploying it on Passenger instead of on runit. Passenger relieves you from even having to write a shell script, and adds all sorts of useful production features like process scaling, introspection, etc. These are not available when you're only using runit.

<a name="selecting_default_ruby"></a>
### Selecting a default Ruby version

The default Ruby (what the `/usr/bin/ruby` command executes) is the latest Ruby version that you've chosen to install. You can use `ruby-switch` to select a different version as default.

    # Ruby 1.9.3 (you can ignore the "1.9.1" suffix)
    RUN ruby-switch --set ruby1.9.1
    # Ruby 2.0
    RUN ruby-switch --set ruby2.0
    # Ruby 2.1
    RUN ruby-switch --set ruby2.1
    # Ruby 2.2
    RUN ruby-switch --set ruby2.2
    # JRuby
    RUN ruby-switch --set jruby

<a name="running_startup_scripts"></a>
### Running scripts during container startup

passenger-docker uses the [baseimage-docker](http://phusion.github.io/baseimage-docker/) init system, `/sbin/my_init`. This init system runs the following scripts during startup, in the following order:

 * All executable scripts in `/etc/my_init.d`, if this directory exists. The scripts are run during in lexicographic order.
 * The script `/etc/rc.local`, if this file exists.

All scripts must exit correctly, e.g. with exit code 0. If any script exits with a non-zero exit code, the booting will fail.

The following example shows how you can add a startup script. This script simply logs the time of boot to the file /tmp/boottime.txt.

    ### In logtime.sh (make sure this file is chmod +x):
    #!/bin/sh
    date > /tmp/boottime.txt

    ### In Dockerfile:
    RUN mkdir -p /etc/my_init.d
    ADD logtime.sh /etc/my_init.d/logtime.sh

<a name="upgrading_os"></a>
### Upgrading the operating system inside the container

passenger-docker images contain an Ubuntu 14.04 operating system. You may want to update this OS from time to time, for example to pull in the latest security updates. OpenSSL is a notorious example. Vulnerabilities are discovered in OpenSSL on a regular basis, so you should keep OpenSSL up-to-date as much as you can.

While we release passenger-docker images with the latest OS updates from time to time, you do not have to rely on us. You can update the OS inside passenger-docker images yourself, and it is recommend that you do this instead of waiting for us.

To upgrade the OS in the image, run this in your Dockerfile:

    RUN apt-get update && apt-get upgrade -y -o Dpkg::Options::="--force-confold"

<a name="upgrading_passenger"></a>
### Upgrading Passenger to the latest version

#### Upgrading to the latest image

Passenger-docker images contain a specific Passenger version by default. We regularly update passenger-docker with the latest version of Passenger, Ruby, Node.js, etc.

To upgrade your image to the latest passenger-docker version, please edit your Dockerfile and change the passenger-docker version in the `FROM` command to the latest version. You can find a list of available versions in the [Changelog](https://github.com/phusion/passenger-docker/blob/master/Changelog.md).

For example, if you were using passenger-docker 0.9.16 and want to upgrade to 0.9.17, then change...

    FROM phusion/passenger-docker-XXXX:0.9.16

...to:

    FROM phusion/passenger-docker-XXXX:0.9.17

Then rebuild your image.

#### Upgrading Passenger without waiting for image updates

We do not update the passenger-docker image on *every* Passenger release. Having said that, you can upgrade Passenger at any time, without waiting for us to release a new image.

Passenger is installed through [the Passenger APT repository](https://www.phusionpassenger.com/library/install/nginx/apt_repo/), so you can use APT to upgrade Passenger.

To upgrade to the latest Passenger version, run this to your Dockerfile:

    RUN apt-get update && apt-get upgrade -y -o Dpkg::Options::="--force-confold"


<a name="container_administration"></a>
## Container administration

One of the ideas behind Docker is that containers should be stateless, easily restartable, and behave like a black box. However, you may occasionally encounter situations where you want to login to a container, or to run a command inside a container, for development, inspection and debugging purposes. This section describes how you can administer the container for those purposes.

_**Tip**: passenger-docker is based on [baseimage-docker](https://github.com/phusion/baseimage-docker). Please consult [the baseimage-docker documentation](https://github.com/phusion/baseimage-docker) for more container administration documentation and tips._

<a name="oneshot"></a>
### Running a one-shot command in a new container

_**Note:** This section describes how to run a command insider a -new- container. To run a command inside an existing running container, see [Running a command in an existing, running container](#run_inside_existing_container)._

Normally, when you want to create a new container in order to run a single command inside it, and immediately exit after the command exits, you invoke Docker like this:

    docker run YOUR_IMAGE COMMAND ARGUMENTS...

However the downside of this approach is that the init system is not started. That is, while invoking `COMMAND`, important daemons such as cron and syslog are not running. Also, orphaned child processes are not properly reaped, because `COMMAND` is PID 1.

Passenger-docker provides a facility to run a single one-shot command, while solving all of the aforementioned problems. Run a single command in the following manner:

    docker run YOUR_IMAGE /sbin/my_init -- COMMAND ARGUMENTS ...

This will perform the following:

 * Runs all system startup files, such as /etc/my_init.d/* and /etc/rc.local.
 * Starts all runit services.
 * Runs the specified command.
 * When the specified command exits, stops all runit services.

For example:

    $ docker run phusion/passenger-full:<VERSION> /sbin/my_init -- ls
    *** Running /etc/rc.local...
    *** Booting runit daemon...
    *** Runit started as PID 80
    *** Running ls...
    bin  boot  dev  etc  home  image  lib  lib64  media  mnt  opt  proc  root  run  sbin  selinux  srv  sys  tmp  usr  var
    *** ls exited with exit code 0.
    *** Shutting down runit daemon (PID 80)...
    *** Killing all processes...

You may find that the default invocation is too noisy. Or perhaps you don't want to run the startup files. You can customize all this by passing arguments to `my_init`. Invoke `docker run YOUR_IMAGE /sbin/my_init --help` for more information.

The following example runs `ls` without running the startup files and with less messages, while running all runit services:

    $ docker run phusion/passenger-full:<VERSION> /sbin/my_init --skip-startup-files --quiet -- ls
    bin  boot  dev  etc  home  image  lib  lib64  media  mnt  opt  proc  root  run  sbin  selinux  srv  sys  tmp  usr  var

<a name="run_inside_existing_container"></a>
### Running a command in an existing, running container

There are two ways to run a command inside an existing, running container.

 * Through the `docker exec` tool. This is builtin Docker tool, available since Docker 1.4. Internally, it uses Linux kernel system calls in order to execute a command within the context of a container. Learn more in [Login to the container, or running a command inside it, via `docker exec`](#login_docker_exec).
 * Through SSH. This approach requires running an SSH daemon inside the container, and requires you to setup SSH keys. Learn more in [Login to the container, or running a command inside it, via SSH](#login_ssh).

Both way have their own pros and cons, which you can learn in their respective subsections.

<a name="login_docker_exec"></a>
### Login to the container, or running a command inside it, via `docker exec`

You can use the `docker exec` tool on the Docker host OS to login to any container that is based on passenger-docker. You can also use it to run a command inside a running container. `docker exec` works by using Linux kernel system calls.

Here's how it compares to [using SSH to login to the container or to run a command inside it](#login_ssh):

 * Pros
   * Does not require running an SSH daemon inside the container.
   * Does not require setting up SSH keys.
   * Works on any container, even containers not based on passenger-docker.
 * Cons
   * If the `docker exec` process on the host is terminated by a signal (e.g. with the `kill` command or even with Ctrl-C), then the command that is executed by `docker exec` is *not* killed and cleaned up. You will either have to do that manually, or you have to run `docker exec` with `-t -i`.
   * Requires privileges on the Docker host to be able to access the Docker daemon. Note that anybody who can access the Docker daemon effectively has root access.
   * Not possible to allow users to login to the container without also letting them login to the Docker host.

<a name="docker_exec_usage"></a>
#### Usage

Start a container:

    docker run YOUR_IMAGE

Find out the ID of the container that you just ran:

    docker ps

Now that you have the ID, you can use `docker exec` to run arbitrary commands in the container. For example, to run `echo hello world`:

    docker exec YOUR-CONTAINER-ID echo hello world

To open a bash session inside the container, you must pass `-t -i` so that a terminal is available:

    docker exec -t -i YOUR-CONTAINER-ID bash -l

<a name="login_ssh"></a>
### Login to the container, or running a command inside it, via SSH

You can use SSH to login to any container that is based on passenger-docker. You can also use it to run a command inside a running container.

Here's how it compares to [using `docker exec` to login to the container or to run a command inside it](#login_docker_exec):

 * Pros
   * Does not require root privileges on the Docker host.
   * Allows you to let users login to the container, without letting them login to the Docker host. However, this is not enabled by default because passenger-docker does not expose the SSH server to the public Internet by default.
 * Cons
   * Requires setting up SSH keys. However, passenger-docker makes this easy for many cases through a pregenerated, insecure key. Read on to learn more.

<a name="enabling_ssh"></a>
#### Enabling SSH

Passenger-docker disables the SSH server by default. Add the following to your Dockerfile to enable it:

    RUN rm -f /etc/service/sshd/down
    
    # Regenerate SSH host keys. Passenger-docker does not contain any, so you
    # have to do that yourself. You may also comment out this instruction; the
    # init system will auto-generate one during boot.
    RUN /etc/my_init.d/00_regen_ssh_host_keys.sh

<a name="ssh_keys"></a>
#### About SSH keys

First, you must ensure that you have the right SSH keys installed inside the container. By default, no keys are installed, so nobody can login. For convenience reasons, we provide [a pregenerated, insecure key](https://github.com/phusion/baseimage-docker/blob/master/image/insecure_key) [(PuTTY format)](https://github.com/phusion/baseimage-docker/blob/master/image/insecure_key.ppk) that you can easily enable. However, please be aware that using this key is for convenience only. It does not provide any security because this key (both the public and the private side) is publicly available. **In production environments, you should use your own keys**.

<a name="using_the_insecure_key_for_one_container_only"></a>
#### Using the insecure key for one container only

You can temporarily enable the insecure key for one container only. This means that the insecure key is installed at container boot. If you `docker stop` and `docker start` the container, the insecure key will still be there, but if you use `docker run` to start a new container then that container will not contain the insecure key.

Start a container with `--enable-insecure-key`:

    docker run YOUR_IMAGE /sbin/my_init --enable-insecure-key

Find out the ID of the container that you just ran:

    docker ps

Once you have the ID, look for its IP address with:

    docker inspect -f "{{ .NetworkSettings.IPAddress }}" <ID>

Now that you have the IP address, you can use SSH to login to the container, or to execute a command inside it:

    # Download the insecure private key
    curl -o insecure_key -fSL https://github.com/phusion/baseimage-docker/raw/master/image/insecure_key
    chmod 600 insecure_key

    # Login to the container
    ssh -i insecure_key root@<IP address>

    # Running a command inside the container
    ssh -i insecure_key root@<IP address> echo hello world

<a name="enabling_the_insecure_key_permanently"></a>
#### Enabling the insecure key permanently

It is also possible to enable the insecure key in the image permanently. This is not generally recommended, but is suitable for e.g. temporary development or demo environments where security does not matter.

Edit your Dockerfile to install the insecure key permanently:

    RUN /usr/sbin/enable_insecure_key

Instructions for logging in the container is the same as in section [Using the insecure key for one container only](#using_the_insecure_key_for_one_container_only).

<a name="using_your_own_key"></a>
#### Using your own key

Edit your Dockerfile to install an SSH public key:

    ## Install an SSH of your choice.
    ADD your_key.pub /tmp/your_key.pub
    RUN cat /tmp/your_key.pub >> /root/.ssh/authorized_keys && rm -f /tmp/your_key.pub

Then rebuild your image. Once you have that, start a container based on that image:

    docker run your-image-name

Find out the ID of the container that you just ran:

    docker ps

Once you have the ID, look for its IP address with:

    docker inspect -f "{{ .NetworkSettings.IPAddress }}" <ID>

Now that you have the IP address, you can use SSH to login to the container, or to execute a command inside it:

    # Login to the container
    ssh -i /path-to/your_key root@<IP address>

    # Running a command inside the container
    ssh -i /path-to/your_key root@<IP address> echo hello world

<a name="docker_ssh"></a>
#### The `docker-ssh` tool

Looking up the IP of a container and running an SSH command quickly becomes tedious. Luckily, we provide the `docker-ssh` tool which automates this process. This tool is to be run on the *Docker host*, not inside a Docker container.

First, install the tool on the Docker host:

    curl --fail -L -O https://github.com/phusion/baseimage-docker/archive/master.tar.gz && \
    tar xzf master.tar.gz && \
    sudo ./baseimage-docker-master/install-tools.sh

Then run the tool as follows to login to a container using SSH:

    docker-ssh YOUR-CONTAINER-ID

You can lookup `YOUR-CONTAINER-ID` by running `docker ps`.

By default, `docker-ssh` will open a Bash session. You can also tell it to run a command, and then exit:

    docker-ssh YOUR-CONTAINER-ID echo hello world

<a name="inspecting_web_app_status"></a>
### Inspecting the status of your web app

If you use Passenger to deploy your web app, run:

    passenger-status
    passenger-memory-stats

<a name="logs"></a>
### Logs

If anything goes wrong, consult the log files in /var/log. The following log files are especially important:

 * /var/log/nginx/error.log
 * /var/log/syslog
 * Your app's log file in /home/app.

<a name="enterprise"></a>
### Switching to Phusion Passenger Enterprise

If you are a [Phusion Passenger Enterprise](https://www.phusionpassenger.com/enterprise) customer, then you can switch to the Enterprise variant as follows.

 1. Login to the [Customer Area](https://www.phusionpassenger.com/orders).
 2. Download the license key and store it in the same directory as your Dockerfile.
 3. Insert into your Dockerfile:

        ADD passenger-enterprise-license /etc/passenger-enterprise-license
        RUN echo deb https://download:$DOWNLOAD_TOKEN@www.phusionpassenger.com/enterprise_apt trusty main > /etc/apt/sources.list.d/passenger.list
        RUN apt-get update && apt-get install -y -o Dpkg::Options::="--force-confold" passenger-enterprise nginx-extras

    Replace `$DOWNLOAD_TOKEN` with your actual download token, as found in the Customer Area.

<a name="building"></a>
## Building the image yourself

If for whatever reason you want to build the image yourself instead of downloading it from the Docker registry, follow these instructions.

Clone this repository:

    git clone https://github.com/phusion/passenger-docker.git
    cd passenger-docker

Start a virtual machine with Docker in it. You can use the Vagrantfile that we've already provided.

    vagrant up
    vagrant ssh
    cd /vagrant

Build one of the images:

    make build_ruby19
    make build_ruby20
    make build_ruby21
    make build_ruby22
    make build_jruby90
    make build_nodejs
    make build_customizable
    make build_full

If you want to call the resulting image something else, pass the NAME variable, like this:

    make build NAME=joe/passenger

<a name="conclusion"></a>
## Conclusion

 * Using passenger-docker? [Tweet about us](https://twitter.com/share) or [follow us on Twitter](https://twitter.com/phusion_nl).
 * Having problems? Please post a message at [the discussion forum](https://groups.google.com/d/forum/passenger-docker).
 * Looking for a minimal image containing only a correct base system? Take a look at [baseimage-docker](https://github.com/phusion/baseimage-docker).

[<img src="http://www.phusion.nl/assets/logo.png">](http://www.phusion.nl/)

Please enjoy passenger-docker, a product by [Phusion](http://www.phusion.nl/). :-)
