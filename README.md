# README

## Docker Compose + Rails

Checkout the [Quickstart: Compose and Rails](https://docs.docker.com/compose/rails/) article.

Basically the process is:

1. Create a [Dockerfile](./Dockerfile)
1. Create a Gemfile with just the Rails gem: `gem 'rails', '~>5'`
1. Create an empty Gemfile.lock file: `touch Gemfile.lock`
1. Create an [entrypoint.sh](./entrypoint.sh) file.
1. Create a basic [docker-compose.yml](./docker-compose.yml) file.
1. Build the project `docker-compose run web rails new . --force --no-deps --database=postgresql`
1. Build the project (again!) to add the rails app to the Docker image (Changes to the Gemfile or the Dockerfile, should be the only times you’ll need to rebuild the image) `docker-compose build`
1. Make sure that the [database config](./config/database.yml) `host`, `username` & `password` is set correctly.
1. Now boot the app! `docker-compose up`
1. Create the database `docker-compose run web rake db:create`
1. Open the Rails Welcome page on [http://localhost:3000](http://localhost:3000)

## Running the database migrations

After you start the app running for the first time you will need to run the database migrations.

Start the app:

```
docker-compose up
```

Enter into a session in the running web container:

```
docker exec -it web bash
```

Now run the db migrations

```
rake db:migrate
```

Of course, you can run rails console, debug etc etc:

```
rails generate scaffold Product title:string description:text image_url:string price:decimal
```

## AWS ECS

Create a new repository for the app (only needs to be done once)

```
aws ecr create-repository --repository-name demo/rails
```

Make a note of the `repositoryUri` (e.g. 924583607971.dkr.ecr.ap-southeast-1.amazonaws.com/demo/rails)

## Create Task Definition to run the Rails App and run DB Migrations

```
aws ecs register-task-definition --cli-input-json ./aws/railsdemo.json
```

## Deploy to AWS Container Services (ECS) using AWS Fargate

When we are ready to deploy we need to:

1. Authenticate with ECR
1. Precompile any assets
1. Build the Docker Image
1. Check the contents of Image
1. Tag the image for ECR
1. Push the Image to ECR
1. Run the database migrations
1. Update the running ECS Service instance.

**Authenticate Docker Client with AWS ECR**

Using `aws` cli, authenticate with the AWS Container Registry (ECR) like so:

```
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 924583607971.dkr.ecr.ap-southeast-1.amazonaws.com
```

**Precompile the assets**

```
SECRET_KEY_BASE=asecret RAILS_ENV=staging bundle exec rake assets:precompile
```

**Build the docker image**

```
docker build -t rails6demo:latest .
```

**(Optional) check the contents of the image**

```
docker run -it rails6demo:latest bash
```

**Tag the image for AWS ECR**

```
docker tag rails6demo:latest 924583607971.dkr.ecr.ap-southeast-1.amazonaws.com/demo/rails:latest
```

**Push to AWS ECR**

```
docker push 924583607971.dkr.ecr.ap-southeast-1.amazonaws.com/demo/rails:latest
```

**Run migrations**

```
aws ecs run-task --cluster railsdemonew --launch-type FARGATE --task-definition railsmmm:7 --network-configuration '{
  "awsvpcConfiguration": {
    "subnets": ["subnet-e10d6796", "subnet-67a6d802"],
    "securityGroups": ["sg-0f23cc29be7d6939e"],
    "assignPublicIp": "ENABLED"
  }
}'
```

**Deploy to service**

The command below will force the running tasks in the service `railsdemoservice` to redeploy with the latest docker image that was just pushed to AWS ECR.

```
aws ecs update-service --cluster railsdemonew --service railsdemoservice --force-new-deployment --region ap-southeast-1
```

## RDS

Note that RDS needs to have the security group to be configured correctly to allow to connect to it from a bastion host or from yoru local machine.

Connect to it using `psql`

```
psql -h hostname -U username database
```

Run migrations:

```
SECRET_KEY_BASE=asecret RAILS_ENV=staging DATABASE_URL=postgres://username:password@hostname/dataabase bundle exec rake db:migrate
```