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

## Deploy to AWS Container Services (ECS) using AWS Fargate

**Install the `aws` cli tool**

Using `aws` cli, authenticate with the AWS Container Registry (ECR) like so:

```
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 924583607971.dkr.ecr.ap-southeast-1.amazonaws.com
```

**Create a repository to push the image to**

```
aws ecr create-repository --repository-name demo/rails
```

Make a note of the `repositoryUri` (e.g. 924583607971.dkr.ecr.ap-southeast-1.amazonaws.com/demo/rails)

**Build the docker image**

```
docker build -t rails6demo:latest .
```

**(Optional) check the contents of the image**

```
docker run -it --entrypoint bash rails6demo:latest
```

**Tag the image for AWS ECS**

```
docker tag rails6demo:latest 924583607971.dkr.ecr.ap-southeast-1.amazonaws.com/demo/rails:latest
```

**Push to AWS**

```
docker push 924583607971.dkr.ecr.ap-southeast-1.amazonaws.com/demo/rails:latest
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