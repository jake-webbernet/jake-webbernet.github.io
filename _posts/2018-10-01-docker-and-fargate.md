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

### Local Docker Compose

I setup my rails application work with Docker Compose. Basically, you want to be able to run `docker-compose up` locally, and have all your resources built, booted and running so you have a working website. In my case, a postgres database and rails application.

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
  
### Setup a Registry
So I have successfully built the image locally and it runs. I need to store this in the cloud using a Docker Image Registry so my servers can access it.

This write up doesn't go into registries to much, but these are the commands used to get it up there.

```
$ docker build -t $(git rev-parse HEAD) .
$ $(aws ecr get-login --region ap-southeast-2 | sed -e 's/-e none//g')
$ docker tag $(git rev-parse HEAD) 123456.dkr.ecr.ap-southeast-2.amazonaws.com/myapp:$(git rev-parse HEAD)
$ docker push 123456.dkr.ecr.ap-southeast-2.amazonaws.com/myapp:$(git rev-parse HEAD)
```

### AWS Docker Compose File

So at the point I was successfully running my little stack locally. I needed to try this out on ECS.

I set out to do some research and actually found that [ECS supports Docker Compose Files](https://aws.amazon.com/about-aws/whats-new/2018/06/amazon-ecs-cli-supports-docker-compose-version-3/). This is great news. It means I can pass my docker compose file onto ECS, and they `will host it all for me!

I need to create another docker compose file that is compatible with AWS 
  
```
version: '3'

services:
  web:
    image: "123456789.dkr.ecr.ap-southeast-2.amazonaws.com/my-app:4f6a07433a35c1369851eacf7773e2cccdc5a651"
    command: ["supervisord", "-c", "/etc/supervisord-staging.conf"]
    ports:
      - 3000:3000
    environment:
      DATABASE_URL: postgres://postgres@localhost
      RAILS_ENV: staging
      RAILS_SERVE_STATIC_FILES: 'true'
    logging:
      driver: awslogs
      options: 
        awslogs-group: /ecs/tutorial
        awslogs-region: ap-southeast-2
        awslogs-stream-prefix: ecs
  db:
    image: postgres:10.3-alpine
    logging:
      driver: awslogs
      options: 
        awslogs-group: /ecs/tutorial
        awslogs-region: ap-southeast-2
        awslogs-stream-prefix: db
```
**Changes**
* Changed version to 3 instead of 3.6 (AWS only supports 3)
* Remove the `build` command and use `image` so I can specify the pre-built image I stored in my registry
* Add the logging parameter. This is so logs are available in CloudWatch and ECS and I can troubleshoot any issues
* Removed the `depends_on` attribute as it's not supported
* Changed the `DATABASE_URL` environment variable to reference `localhost` instead of `db`
  * Previously our `depends_on` link was setting up a host called `db`, and we were relying on that in the database URL to connect our app to the database
  * In Fargate we are forced to utilise a networking method called [awsvpc](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking.html) which allows our containers to communicate over the `localhost` interface instead

### Provisioning Resources
So most of preperation work is completed and it's time to connect with AWS

I used this [AWS Tutorial](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_tutorial_fargate.html) to create my resources

Quick Overview of what this tutorial goes through
* Using the `ecs-cli` CLI interface
* Creating an IAM role that our servers can use, and allowing them to speak to ECS via a trust policy
* Creating a VPN with two public subnets for our applications to reside
* Creating a VPN security group that opens port 80
* Creating a `ecs-params.yml` file with networking information regarding the VPN
* Creating a cluster
* Creating a service


### Gotchas
* There does not seem to be a way to control order in which services are provisioned in ECS. Sometimes my database would not be finished provisioning, and so my app was not able to run setup commands on the database and tables were missing!
  * I had to manually add `sleep` commands into the `rake db:migrate` commands on my app which is a bit hacky and not scaleable.
* The networking mechanism form the `awsvpc` tripped me up a bit. Having the two services communicate.

