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

## Building an App

Some quick notes on building the app

```
rails generate scaffold Product title:string description:text image_url:string price:decimal
```