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

- clone the repo
- install docker
- fill in secret keys
- run the repo with `docker-compose up` to view all the logs
    - include `-d` to run the containers detached
- go to `localhost:9000` in a web browser
