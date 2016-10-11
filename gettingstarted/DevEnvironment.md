# Setting up the development environment
Before you can start contributing to the official Zetkin applications, you need
to set up the development environment.

The Zetkin Foundation uses Docker for development, to cleanly separate the
applications and their requirements from the host system.

## Quickstart
If you are mostly familiar with Docker, npm, gulp, networking and the Zetkin
authentication workflow, you might be able to skip right down to the _Summary_
section of this document.

If you feel unsure about any of this, please keep reading the deatailed
walk-through.

## Docker basics
Docker is a technology which allows applications to run within _containers_,
which are instances of _images_ that define the entire system in which the apps
run. Images can be described in a _Dockerfile_ which defines all the steps that
are necessary to re-create the system.

All Zetkin apps come with a Dockerfile that sets up all prerequisites and runs
the application. This means that to get started with Zetkin development, all you
need is Docker, and the rest will be installed in a self-contained environment
that does not affect your main system.

If you do not already have Docker on your system, follow the official
[instructions on Docker's website](https://www.docker.com/products/overview) to
install Docker on your Mac, Windows or Linux system.

Docker can be run in a virtual machine using VirtualBox. This effectively
separates the Docker sandbox completely from your host system. By default,
Docker is run directly on your host. Among other things, this means that if you
have a web server running on port 80 on your machine, this will prevent Docker
from using that port. You can temporarily stop your regular web server, or use
[Docker Machine](https://docs.docker.com/machine/overview/) to run Docker in a
VirtualBox virtual machine.

__NOTE__: If you are using Docker Machine, there seems to be an issue related
to DNS after moving between networks (as is common with a laptop). If you plan
on running Zetkin inside Docker Machine, before you do anything, restart your
virtual machine using `docker-machine restart default`, where "default" is the
name of your machine.

## The Zetkin development instance
For the convenience of Zetkin developers, the Zetkin Foundation maintains a
development instance of Zetkin running at _dev.zetkin.org_. This instance is
fully functional but completely separate from regular Zetkin at zetk.in.

Because it's completely separate, it does not share user accounts, registered
third-party applications or any other data with regular Zetkin.

When contributing to official Zetkin applications, the development instance is
a good way to not have to host the entire Zetkin eco-system (parts of which
are not open-source or available outside of Zetkin Foundation). Instead, you
only have to run the application you're working on.

## Setting up the environment
Setting up the environment to contribute to an official Zetkin application can
be broken down to a few steps.

1. Get the source code.
2. Replicate the app environment on your local machine using Docker.
3. Hook up your local app to the Zetkin development instance at dev.zetkin.org.
4. Make changes to the local code and test them in your new environment.

Lets look at a case where you want to contribute to the Zetkin activist portal
(_www.zetk.in_) by slightly changing the style.

### 1. Get the source code
Code for the open-source official Zetkin applications is available on GitHub.
To contribute back to the official repository, you need to be able to create
pull requests, which means that you should generally create your own fork of
the application you intend to work on. Eventually, you might be trusted with
write access to the repository, at which point you no longer need your own fork
but can instead work in local branches.

For now, lets assume you have created your own fork, and that your GitHub user
is richardolsson. All you need to do is find a suitable location on your drive
and clone your fork of the application repository.

```
$ git clone git@github.com:richardolsson/www.zetk.in.git
$ cd www.zetk.in
```

This creates a local copy of the code. If you're not familiar with Git, please
first read the Git tutorial and come back here to continue once you feel ready.

### 2. Set up application environment
The application contains a Dockerfile which describes the environment it needs
to run. You need to build a Docker image using this Dockerfile. Furthermore,
this is the time to install any node dependencies and create an initial build
of the source.

Assuming that you are not using Docker Machine, but instead the most
simple Docker set up, building the image is just a single Docker command.

```
$ docker build -t www_zetk_in env/app
```

Because www.zetk.in is a Node app, you also need to install npm dependencies
and build the source code using Gulp. If you have npm and gulp pre-installed
in your system, just do the following.

```
$ npm install
$ gulp
```

If you do not have npm and gulp, you can install it, or if you prefer to
keep your system completely clean, run it from within the Docker container, but
that is left as an excercise for the reader for now.

### 3. Running and connecting it all
The next step is to run the Docker image inside a container. The following
Docker command creates a container from the recently created `www_zetk_in`
image, calls the container by that same name, sets a bunch of required variables
which the application uses to connect to the correct (development) instance
and listens for incoming connections on port 80.

The app ID and key vary from application to application. Talk to the project
managers if you need the correct values for another app than www.zetk.in.

```
$ docker run -d -v $PWD:/var/app --name www_zetk_in \
    --env ZETKIN_LOGIN_URL=http://login.dev.zetkin.org \
    --env ZETKIN_APP_ID=a4 \
    --env ZETKIN_APP_KEY=ghi789 \
    --env ZETKIN_DOMAIN=dev.zetkin.org \
    -p 80:80 www_zetk_in
```

Running this command should start the application and start serving it at
http://localhost:80. You can browse to that URL, but because the application
requires a user to be authenticated, it will instantly redirect you to the
Zetkin login page at http://login.dev.zetkin.org.

This is perfectly fine, except once you've logged in, you will be redirected to
http://www.dev.zetkin.org (because that's the callback that has been
configured for the app in the Zetkin application registry). This is a security
feature, that prevents other applications from redirecting you to the login page
and getting hold of your ticket once you return.

To be able to log into your local copy, you need to configure your computer to
request your local copy instead of the one running on the development instance.
This is most easily done by modifying the system's hosts file, so that the
system resolves www.dev.zetkin.org to localhost.

Edit the hosts file (/etc/hosts on Mac or Linux) and make sure the following
line is present (and has not been commented out with an initial `#`).

```
127.0.0.1   www.dev.zetkin.org
```

Clear your browser and DNS cache and try again browsing to
http://www.dev.zetkin.org. You will be redirected to the login page (even
if you signed in earlier) and after logging in you should be redirected back to
your local copy of the application.

If logging in does not work, manually browse to /logout in the application,
which for all official Zetkin applications clears cookies and other possible
remnants of the original environment.

NOTE: Make sure you remove or comment out the line in your hosts file when you
no longer want to use your local copy, but the real dev version instead.

### 4. Making changes to the code
Once you've come this far you are ready to make changes to the code. You can
try editing a SASS file to add a new background color to the body element.

After making such a change, the application will need to be rebuilt using gulp,
and for some changes (especially to javascript code) even restarted within the
container.

```
$ gulp
$ docker exec www_zetk_in sv restart app
```

The first command rebuilds, and the second restarts the application in the
container. The exact commands vary from application to application.

Oftentimes, there is a gulp task called "watch" which watches the source code
for changes, rebuilds the necessary code and restarts the project. All you need
is to run `gulp watch` with an environment variable set to indicate the name of
the Docker container in which the application is running.

```
$ ZETKIN_CONTAINER_NAME=www_zetk_in gulp watch
```

Once gulp watch is running, edit any file and watch how the code is rebuilt and
the application restarted automatically. Just refresh the browser to see the
changes.

## Summary
Below is a summary of the commands executed, in order, to get everything up and
running. Depending on what app you are trying to get running, whether you're
using Docker Machine or not, this may or may not be exactly what you need.

```
$ git clone git@github.com:richardolsson/www.zetk.in.git
$ cd www.zetk.in
$ docker build -t www_zetk_in env/app
$ npm install
$ gulp
$ docker run -d -v $PWD:/var/app --name www_zetk_in \
    --env ZETKIN_LOGIN_URL=http://login.dev.zetkin.org \
    --env ZETKIN_APP_ID=a4 \
    --env ZETKIN_APP_KEY=ghi789 \
    --env ZETKIN_DOMAIN=dev.zetkin.org \
    -p 80:80 www_zetk_in
$ sudo sh -c 'echo "127.0.0.1 www.dev.zetkin.org" >> /etc/hosts'
$ ZETKIN_CONTAINER_NAME=www_zetk_in gulp watch
```
