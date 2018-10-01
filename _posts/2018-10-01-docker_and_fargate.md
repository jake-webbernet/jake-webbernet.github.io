# ECS Fargate and Docker #

I've used Docker for mostly deployment related tasks, however I wanted to dig a little bit deeper with Docker Compose.

## Goals

IN
* I want to easily be able to spin up branches of our applications for short periods of time.
* This should enable customers to play with certain branches of code in a seperate, repeatable environment
* This should enable our QA team to test code without needing to do anything technical on their end (just open a website)
* Applications should be seeded, meaning there is fake data already in the database

OUT
* Allowing the Development team to use the docker compose setup to develop locally

## Tech

### Docker Compose
It allows you to orchestrate multiple docker images together using a `docker-compose.yml` file.

### ECS (Specifically, Fargate)
Our deployment system of choice. At the moment we use EC2 launch types but I think Fargate is a better option

* Fargate seems to be more suited to the 'temporary workload' type of work (our servers won't be serving 24/7)
* Fargate allows me to not need to worry about provisioning EC2 instances. Great! One less thing I need to scale

## What I did

1. I setup my rails application work with Docker Compose. Basically, you want to be able to run `docker-compose up` locally, and have all your resources built, booted and running so you have a working website. In my case, a postgres database and rails application.

Here is an example of my local docker-compose file

```
version: '3.6'

services:
  web:
    build: .
    command: ["supervisord", "-c", "/etc/supervisord-staging.conf"]
    ports:
      - 3000:3000
    depends_on:
      - db
    environment:
      DATABASE_URL: postgres://postgres@db
      RAILS_ENV: staging
      RAILS_SERVE_STATIC_FILES: 'true'
  db:
    image: postgres:10.3-alpine
```

  * Notice the `build: .` command. This means when running `docker-compose up` the local application would be built into a docker image and run.

2. Created a secondary docker-compose file






