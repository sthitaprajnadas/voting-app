# Voting App with Docker (Node.js version)

<!--
## Build Status

| Service  | Docker Image           | Build Status |
|:---------|:-----------------------|:-------------|
| API      | subfuzion/vote-api     | [![Docker build](https://img.shields.io/docker/build/subfuzion/vote-api.svg)](https://hub.docker.com/r/subfuzion/vote-api/)
| Worker   | subfuzion/vote-worker  | [![Docker build](https://img.shields.io/docker/build/subfuzion/vote-worker.svg)](https://hub.docker.com/r/subfuzion/vote-worker/)
| Auditor  | subfuzion/vote-auditor | [![Docker build](https://img.shields.io/docker/build/subfuzion/vote-auditor.svg)](https://hub.docker.com/r/subfuzion/vote-auditor/)
| Database | Mongo | [![Docker build](https://img.shields.io/docker/pulls/_/mongo.svg)](https://hub.docker.com/_/mongo/)
| Queue | Redis | [![Docker build](https://img.shields.io/docker/pulls/_/redis.svg)](https://hub.docker.com/_/redis/)

| Node.js Packages    | npm                    | Build Status |
|:--------------------|:-----------------------|:------------ |
| @subfuzion/database | [![npm (scoped)](https://img.shields.io/npm/v/@subfuzion/database.svg)](@subfuzion/database) | [![Travis](https://img.shields.io/travis/subfuzion/voting-app.svg)](https://travis-ci.org/subfuzion/voting-app)
| @subfuzion/queue    | [![npm (scoped)](https://img.shields.io/npm/v/@subfuzion/queue.svg)](@subfuzion/queue) | [![Travis](https://img.shields.io/travis/subfuzion/voting-app.svg)](https://travis-ci.org/subfuzion/voting-app)

-->

## Quick Start

Install [Docker](https://docs.docker.com/get-docker/).
This app will work with versions from either the Stable or the Edge channels.

> If you're using [Docker for Windows](https://docs.docker.com/docker-for-windows/) on Windows 10 pro or later, you must also switch to [Linux containers](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).

In the project's root directory, run:

    $ docker-compose up

You can test it with the `voter` CLI:

```
$ docker run -it --rm --network=host subfuzion/voter vote
? What do you like better? (Use arrow keys)
  (quit)
❯ cats
❯ dogs
```

You can print voting results:

```
$ docker run -it --rm --network=host subfuzion/voter results
Total votes -> cats: 0, dogs: 1 ... DOGS WIN!
```

When you are finished:

Press `Ctrl-C` to stop the stack, then enter:

    $ docker-compose -f docker-compose.yml rm -f

### Docker Swarm

You can also run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/).
If you haven't initialized one yet, run:

    $ docker swarm init

Once you have initialized a swarm, then deploy the stack:

    $ docker stack deploy --compose-file docker-stack.yml vote

You can test it the same way as described for docker-compose. When finished, you
can stop the stack by entering:

    $ docker stack rm vote

## About the Voting App

![Voting app architecture](https://raw.githubusercontent.com/subfuzion/voting-app/master/images/voting-app-arch-1.1.png)

This app is based on the original [Docker](https://docker.com) [Example Voting App](https://github.com/dockersamples/example-voting-app).

For an orientation, see this [presentation](http://bit.ly/voting-app-with-docker).

## License

The Voting App is open source and free for any use in compliance with the terms of the
[MIT License](https://github.com/subfuzion/voting-app/blob/master/LICENSE).




# SPD

## Deploying in fargate using Amazon Linux (https://medium.com/containers-on-aws/deploy-the-voting-app-to-aws-ecs-with-fargate-cb75f226408f)
install docker (sudo yum update -y  ,  sudo amazon-linux-extras install docker , sudo service docker start , sudo usermod -a -G docker ec2-user , docker info)
aws configure

    $(aws ecr get-login --no-include-email --region us-east-1)

    $ aws ecr create-repository \
    --repository-name worker \
    --image-scanning-configuration scanOnPush=true \
    --region us-east-1

    $ aws ecr create-repository \
    --repository-name voteapi \
    --image-scanning-configuration scanOnPush=true \
    --region us-east-1

    

### push worker image
- Changed node:9 to node:10.6 in Dockerfile

        $ cd $VOTEAPP_ROOT/src/worker
        $ docker build -t worker .

  docker tag worker:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/worker:latest
    
        $ docker tag worker:latest 437637487786.dkr.ecr.us-east-1.amazonaws.com/worker:latest        
        $ docker push 437637487786.dkr.ecr.us-east-1.amazonaws.com/worker:latest

    
### push voteapi image
        $ cd $VOTEAPP_ROOT/src/vote    
        $ docker build -t voteapi .
        $ docker tag voteapi 437637487786.dkr.ecr.us-east-1.amazonaws.com/voteapi
        $ docker push 437637487786.dkr.ecr.us-east-1.amazonaws.com/voteapi

    
 aws cloudformation deploy --stack-name=voteapp --template-file=aws/voteapp.yml --capabilities=CAPABILITY_IAM
    
 aws cloudformation deploy --stack-name redis --template-file=aws/redis.yml
    
 aws cloudformation deploy --stack-name worker --template-file=aws/worker.yml --parameter-overrides \
 ServiceName=worker ImageUrl=437637487786.dkr.ecr.us-east-1.amazonaws.com/worker:latest DesiredCount=1 \ MongoUri="mongodb+srv://service:Password1@voteapp.gylxj.mongodb.net/myFirstDatabase?retryWrites=true&w=majority"   
 
 aws cloudformation deploy --stack-name voteapi --template-file=aws/voteapi.yml --parameter-overrides \
 ServiceName=voteapi ImageUrl=437637487786.dkr.ecr.us-east-1.amazonaws.com/voteapi:latest ContainerPort=3000 \
 DesiredCount=1 MongoUri="mongodb+srv://service:Password1@voteapp.gylxj.mongodb.net/myFirstDatabase?retryWrites=true&w=majority"
    
    
    
MongoDB URL - mongodb+srv://service:Password1@voteapp.gylxj.mongodb.net/myFirstDatabase?retryWrites=true&w=majority    
    
    
    
export VOTEAPI='http://votea-publi-d7ba4g6j14ok-918299899.us-east-1.elb.amazonaws.com/'
$ cd $VOTEAPP_ROOT/src/voter
$ docker build -t voter .
$ alias voter="docker run -it --rm -e VOTE_API_URI=$VOTEAPI voter"
    
    
### Cleanup
aws cloudformation delete-stack --stack-name voteapp
