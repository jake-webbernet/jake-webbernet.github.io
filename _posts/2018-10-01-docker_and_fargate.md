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
Our deployment system of choice. At the moment we use EC2 launch types but 





