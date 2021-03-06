# Simpl.world Website

[![Contributors](https://img.shields.io/github/contributors/simplworld/simpl.world.website.svg)](https://github.com/simplworld/simpl.world.website/graphs/contributors)

The [simpl.world](https://simpl.world/) website is a static site compiled with [Jekyll](https://jekyllrb.com/docs/home/).

## To Install / Run Locally

To run Jekyll locally (see `Procfile` to see what's going on):

```shell script
$ gem install jekyll bundler foreman
$ foreman start
# => Now browse to http://localhost:8000
```

To manually run Jekyll sans foreman:

```shell script
$ bundle exec jekyll serve
# => Now browse to http://localhost:4000
```

If you get GemNotFound errors from either command, you may need to also run:

```shell script
$ bundle install
```

## To test the site (before deploying)
   
```shell script
$ bundle exec rake test
```

## To Run Jekyll via Docker

We are using [Docker Compose](https://docs.docker.com/compose/) which 
requires [installing it for the platform of your choice](https://docs.docker.com/compose/install/).

```shell script
$ docker-compose pull
$ docker-compose up --build
# => Now browse to http://localhost:8000
```

## For Mac OS High Sierra 

Install ruby using homebrew, add to .bash_profile:

```shell script
# Ruby exports
export GEM_HOME=$HOME/gems
export PATH=$HOME/gems/bin:$PATH

# Add Homebrew's executable directory to the front of the PATH
export PATH=/usr/local/bin:$PATH
```

then run:

```shell script
$ sudo gem install -n /usr/local/bin bundler jekyll foreman
$ foreman start
# => Now browse to http://localhost:8000
```

If you run into gem version errors, these may clear them:

```shell script
$ gem pristine commonmarker --version 0.17.9
$ gem install rake && bundle install
```