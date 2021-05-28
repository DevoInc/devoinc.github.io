# Devo engineering blog

## How to customize the template

This blog is configured to use the `minimal-mistakes` template, which is quite customizable.

To know more about the customizations that can be done, go to:

- [General instructions](https://mmistakes.github.io/minimal-mistakes/docs/configuration/).
- [Override the theme defaults](https://mmistakes.github.io/minimal-mistakes/docs/overriding-theme-defaults/).
  Default templates have already been copied on this repository at /includes.
- [Play with SASS](https://mmistakes.github.io/minimal-mistakes/docs/stylesheets/).
  This repository already includes a copy of assets/css/main.scss that can be modified.

## How to build the static site

If you have Jekyll installed locally (see [the official doc](https://jekyllrb.com/docs/installation/))
you can execute:

```sh
bundle exec jekyll build
```

Otherwhise you can do it with docker.
It will be a little bit slower, but you don't need to modify your local machine:

```sh
docker run \
  -it \
  --rm \
  --volume="$PWD:/srv/jekyll" \
  jekyll/jekyll:3.8 jekyll build
```

The resulting static site will be under `docs/`.

## How to publish on GitHub

The blog uses some plugins that are not supported by GitHub pages.
Therefore, we need to build Jekyll pages locally and then commit the `docs/`
directory on each modification.

## How to run locally

### Using Docker

```sh
docker run -it --rm --volume="$PWD:/srv/jekyll" --env JEKYLL_ENV=development -p 4000:4000 jekyll/jekyll:3.8 jekyll serve
```

## Without Docker

These two steps only the first time:

1. Follow the instructions listed [here](https://jekyllrb.com/docs/installation/).
  (To install it on Mac, follow [this instructions](https://jekyllrb.com/docs/installation/macos/).)
1. On this folder, run `bundle add webrick`. In fact this is only needed if using Ruby 3, but shouldn't hurt in other cases.

Run `jekyll serve` (or perhaps `bundle exec jekyll serve`).

After a while, the console will said something like:

```
Server address: http://0.0.0.0:4000/   
Server running... press ctrl-c to stop.
```

## Open the local version

Then you can open your browser and navigate to [`localhost:4000`](http://localhost:4000/) to open your local version of the blog.
