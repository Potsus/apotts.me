# apotts.me skeleton
---

## The Problem:
I've written code in several different languages that need to be served up in different ways. I need a webserver that can route requests to a single domain to these different projects. 

## The Goal:
create a portable server that can run snippets of code i've worked on in the past as well as new projects from one web address. It's clear that I'll need to use a general purpose webserver to handle the domain, either NGINX or APACHE for now, and route certian urls to different applications. I'm hoping i can get away with using directories for different projects rather than subdomains.

## The Structure:
- Routing: NGINX
- Service seperation: Docker
- Static Pages: Jekyll
- FastFig pages: PHP, node, python
- Resume: Jekyll
- REST API: Tornado? Django

As I see it now I'm looking at using docker containers to seperate out the different projects and ensure they stay running, bringing them back up if they error out or fail tests.

The file structure might look like this:

```
apotts.me
    - keys
        ssh
        whatever

    start.sh
    nginxenv.template #contains the environment variables for nginx

    - nginx docker
    - projects (each in a docker image to expose endpoints)
        - php
            Dockerfile

        - node
            Dockerfile

        - django
            Dockerfile

        - flask
            Dockerfile

        - GeoSlice
            Dockerfile

        - AppInspiration
            Dockerfile

        - BrowserCheck
            Dockerfile

    - Jekyll
        Dockerfile
```


## Ideas:
How am I going to puth this together and what are my goals when designing the structure
Wants:
- Centralized css/sass
- Centralized layout stuff
- Automatic project discovery and routing
- Containers restart if they fail tests
- Portability: able to run just as well on a Raspberry Pi as an aws instance

### Look and Feel structure
1. Have all projects output html that I insert as the body of a template at the top level.
Pros: bad idea
Cons: Seems like a lot of work and loses easy customization and reusability, i have to have every page as an api or something

2. Each project runs in an Iframe on a static page
Pros: relatively easy to setup while maintaining portability, preserves root site templating
Cons: can potentially lead to clashing language and layout edge cases galore

3. Each project gets routed directly and uses it's own templates and css
Pros: probably the easiest and least error prone
Cons: loses cohesive style, loses overall site navigation

Probably the best move is to start with #3 to get it working properly, then move to #2 and only have a top and bottom bar wrap content

### Key Storage and Security
I want to centralize the key storage so that each server can properly negotiate https and whatnot

1. Have the root webserver negotiate all encryption and then pass off requests
Pros: straightforward, seperation of concerns makes sense
Cons: not sure yet, reusability might be a concern

2. Have each project source the same credentials in setup and negotiate their own connections
Pros: maybe more portable?
Cons: could get messy, have to setup a way to acquire creds for each project

Probably just going to go with #1

### Project Discovery
Find a way to discover new projects and route to them if they pass tests

1. Build it yourself:
- List the folders in 'projects'
- check if the folder has a docker instance
- start it and make sure it passes tests
- create a route with the project name as the directory pointing at the exposed docker service
- (later on) create a static page with the site template and and iframe to the project
- add the route to the static sitemap


# Getting Started
---

first install docker


# Notes
---

### Docker
Dockerfiles are good for specifying exactly what one image should look like to docker
docker-compose.yml is for composing multiple docker instances into a coherent app

We're going to use a dockerfile for each project and then compose them all at once so we can bring the entire site up with `docker-compose up`


The nginx docker image we are using also contains [NGINX Amplify](https://github.com/nginxinc/docker-nginx-amplify)

in order to use this properly we need to set a couple of environment varaiables on startup, namely:
- API_KEY: our amplify api key
- AMPLIFY_IMAGENAME: a name for the instance in amplify, consider changing it for prod or to differentiate between instances

#### Attaching to an instance
list all running docker instances with `docker ps`
list all images using `docker images`

you can attach to the stdout of a container using `docker attach`
you can get a terminal in a running instance using `docker exec -it <container name> bash` where bash is the command to run on the container


### Nginx
config files in the container are located in `/etc/nginx/`
log files are at `/var/log/nginx/`

The command to start nginx in `docker-compose.yml` will also substitute environment variables from `mysite.template`. I'm not sure if i'll need this yet, ssh keys? 

### Jekyll
This is where all of my static files are generated.
The generated files in `/Jekyll/_site` are mounted to the Nginx docker container in `docker-compose.yml` and can be modified.
Running `bundle exec jekyll build --watch` from the jekyll directory will auto rebuild the site whenever changes are detected


### Git Submodules
add another git repo as a module with `git submodule add <github link> <submodule name>`
remove a submodule with:
```
git submodule deinit -f -- a/submodule    
rm -rf .git/modules/a/submodule
git rm -f a/submodule
```




