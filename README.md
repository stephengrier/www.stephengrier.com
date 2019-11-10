# Jekyll source of my Github Pages site

This repo contains the source for my blog site. The site is generated with
[Jekyll](https://jekyllrb.com/) and deployed to [Github
Pages](https://pages.github.com/).

## Theme

The site makes use of the [Reverie](https://github.com/amitmerchant1990/reverie)
Jekyll theme, which is simple and clean and suitable for a simple blog.

## Testing locally

The site is deployed to Github Pages. However, it can be viewed locally using
Jekyll. Jekyll is distributed as a ruby gem and so can be installed along with
its dependencies like so:

```
$ gem install bundler
$ bundle install
```

You can then view the site using jekyll:

```
$ bundle exec jekyll serve
$ open http://127.0.0.1:4000/
```
