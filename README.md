# README

## Docker Compose + Rails

Checkout the [Quickstart: Compose and Rails](https://docs.docker.com/compose/rails/) article.

Basically the process is:

1. Create a [Dockerfile](./Dockerfile)
1. Create a Gemfile with just the Rails gem like so:

```
source 'https://rubygems.org'
gem 'rails', '~>5'
```

1. Create an empty Gemfile.lock file: `touch Gemfile.lock`
1. Create an entrypoint.sh file like so:

```
#!/bin/bash
set -e

# Remove a potentially pre-existing server.pid for Rails.
rm -f /myapp/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"
```

1. Create a basic [docker-compose.yml](./docker-compose.yml) file.