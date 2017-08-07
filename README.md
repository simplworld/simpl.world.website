# Simpl.world Website

The www.simpl.world website is a static site compiled with [Jekyll](https://jekyllrb.com/docs/home/).

## To Install / Run Locally

To run Jekyll locally (see `Procfile` to see what's going on):

```shell
$ gem install jekyll bundler foreman
$ foreman start
# => Now browse to http://localhost:8000
```

To manually run Jekyll sans foreman:

```shell
$ bundle exec jekyll serve
# => Now browse to http://localhost:4000
```
