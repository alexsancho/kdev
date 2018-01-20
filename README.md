# kdev

## What is this for?

Kdev is a helper script for all your container related development needs. It's for locally developing containerized applications.

## Quick installation

Install kdev dependencies and start the development environment by running:

    $ curl -fsSL https://raw.githubusercontent.com/alexsancho/kdev/master/bin/bootstrap | bash

If you're installing on a Ubuntu machine, run:

    $ curl -fsSL https://raw.githubusercontent.com/alexsancho/kdev/master/bin/ubuntu | bash

When using linux and MacOS computers on a shared project and you have different docker-compose.yml files for each, linux users need to add an environment variable for kdev to be able to use the correct compose configuration. Add this to your chosen shell configuration (for example .bashrc or .zshrc).

    $ # Export compose file for ubuntu
    $ export COMPOSE_FILE="docker-compose-ubuntu.yml"

## Start project containers

Start a new shell and cd to a project that uses [docker-compose.yml](https://docs.docker.com/compose/)

```
$ kdev up
```

## What this tool includes
kdev installer installs docker for mac plus few settings for better development environment.
It setups 4 service containers into your docker.

### Service Containers

#### Nginx proxy
This project uses excellent nginx proxy container from [jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy). This proxy reserver ports http/https ports from your localhost and proxies the requests to your project containers. Just provide `VIRTUAL_HOST=your-address.local` env in your projects `docker-compose.yml` and nginx proxy will take care of the rest.

#### Custom DNS server
We want to use real addresses for all containers. Some applications have strange behaviour if they are just used from `localhost:8080`. We use [andyshinn/dnsmasq](https://github.com/andyshinn/dnsmasq) for local dnsmasq which always responds `127.0.0.1` to any request. Installation script adds custom resolver file for your machine:

```
$ cat /etc/resolver/local
domain local
nameserver 127.0.0.1
search_order 1
```

This means that all `*.local` addresses are now pointing into your local machine so you don't have to edit `/etc/hosts` file ever again. We used `.local` tld because [it is reserved by IETF](https://en.wikipedia.org/wiki/.local) and will never be [sold to google](http://www.theregister.co.uk/2015/03/13/google_developer_gtld_domain_icann/) like what happened to it's popular cousin `.dev`.

#### Custom https certificate generator
It's a really good practise to use https in production but only a few people use it in development. This makes it harder for people to notice `mixed content` error messages in development.

While using kdev you won't see any of these:

![non-trusted https](https://cloud.githubusercontent.com/assets/5691777/13670188/1b042b48-e6d1-11e5-804e-542781b85ff5.png)

and instead more of these:

![self trusted https](https://cloud.githubusercontent.com/assets/5691777/13670189/1d697032-e6d1-11e5-99b5-aef757cb7f53.png)

kdev includes custom certificate generator [onnimonni/signaler](https://github.com/onnimonni/signaler). kdev installer creates a self-signed unique ca certificate during installation and saves it in your system keychain. If you provide `HTTPS_HOST=your-address.test` env in your `docker-compose.yml` you will automatically have self-signed and trusted certificate for your development environment.

#### Custom SMTP server for debugging email
We included [mailhog/mailhog](https://hub.docker.com/r/mailhog/mailhog/) docker container for easier debugging of emails. Just use `172.17.0.1:25` as smtp server in your application and you are good to go.

Or if your legacy application has hard coded email server you can use this trick in your `docker-compose.yml`:

```
extra_hosts:
    - "my-mail-server.com:172.17.0.1"
```

and docker will add it into the `/etc/hosts` file inside the container automatically during startup.


## Short list of usual commands

```
# This is similiar to vagrant up
# It reads docker-compose.yml from current directory and starts up containers
$ kdev up

# Open shell into web container
$ kdev shell

# Restart all project containers
$ kdev reload

# List all containers from project
$ kdev ps

# List all containers from docker
$ docker ps -a

# Open bash into any container
$ docker exec -it $CONTAINER_ID bash

# Create new project (interactive wizard for setting up project)
$ kdev create
```

## Creating new project
Before creating a new project you should setup a GIT repository for your new project.

It's also advisable to create a config file to your home directory with some default values. File should be named ~/.kdev/kdevconf.yml

Example kdevconf.yml:
```
create:
  defaults:
    wordpress:
      smtp_host: "172.17.0.1"
      theme: "git@github.com:alexsancho/wp-starter-dustpress-theme.git"
```

## Workflow

- The source code running inside a project container is loaded from the directory on your hard drive. You can use text editors and Git clients on the host machines, and shouldn't need to work in the guest machine or the container.
- You should not need to run any application code directly from your host machine. Try to force yourself to find a containerized way of accomplishing things.
- Run `kdev` without any arguments for lots of help

### Troubleshooting

#### No space left on device
Docker for Mac has only limited amount of disk space and this means that older images or stopped containers are taking all of the 60gb/120gb share.

To resolve this delete stopped containers, dangling images and dangling volumes. This can be done by running cleanup helper:

```
$ kdev cleanup
```

If Docker for Mac still has a bug with freeing up disk space, dump databases you need and reset Docker for Mac settings. This will free all the space Docker is hogging. Then you will need to set up your projects again (import databases).

#### When in doubt, update and restart everything

To update all containers and settings run following global commands:
```
$ kdev pull
$ kdev update
$ kdev service pull
$ kdev service build
$ kdev service reload
```

Then restart docker for mac and run these commands:

```
# Reload project containers
$ cd /to/your/project
$ kdev reload
```

## License

`kdev` is available under the MIT license. See the LICENSE file for more info.

Copyright 2018 Alex Sancho.
