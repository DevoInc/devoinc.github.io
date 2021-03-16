# Devo engineer blog

## How to customize the template
This blog is configured to use the `minimal-mistakes` template, which is quite customizable.

To know more about the customizations that can be done, go to:

- [General instructions](https://mmistakes.github.io/minimal-mistakes/docs/configuration/).
- [Override the theme defaults](https://mmistakes.github.io/minimal-mistakes/docs/overriding-theme-defaults/).
  - Default templates have already been copied on this repository at /includes.
- [Play with SASS](https://mmistakes.github.io/minimal-mistakes/docs/stylesheets/).
  - This repository already includes a copy of assets/css/main.scss that can be modified.

## How to run locally
### Start the jekyll
#### Using Docker
On this folder, run

```docker run -it --rm --volume="$PWD:/srv/jekyll" --env JEKYLL_ENV=development -p 4000:4000 jekyll/jekyll:3.8 jekyll serve```

Therefore you can open your browser and navigate to [http://localhost:4000/] to open your local version of the blog.

#### Installing on your machine

1. Follow the instructions listed [here](https://jekyllrb.com/docs/installation/).
    - Specifically, to install it on Mac, follow [this instructions](https://jekyllrb.com/docs/installation/macos/).
2. On this folder, run ```jekyll serve```

### Open the blog
After a while, the console will said something like:

``` 
    Server address: http://0.0.0.0:4000/   
    Server running... press ctrl-c to stop.
```

You can just open [http://localhost:4000](http://localhost:4000) on your browser and the blog should be there.
