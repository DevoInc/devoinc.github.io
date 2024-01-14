---
title: "Node.js &amp; npm best practices"
author: Antonio Olmo Titos
categories: 
  - Node.js
last_modified_at: 2023-11-15T00:00:00Z
type: posts
header:
  teaser: /assets/images/posts/nodejs-and-npm-best-practices/teaser.jpeg
tags:
  - JavaScript
  - JS
  - Node.js
  - npm
  - best practices
classes: wide
---

Perhaps you would be surprised to learn that here at Devo we were early adopters of [Node.js](https://nodejs.org/).

Two of our core propositions and unique features from day one are raw volume of data, and speed in retrieval and processing.
Customers know that well: blazingly fast queries over huge data lakes is one of the features that set us
apart from our competitors.
At the same time, the JavaScript ecosystem is not widely praised for its performance&nbsp;&mdash;&nbsp;admittedly,
there are languages that have been much &ldquo;closer to the metal&rdquo; for ages.
Those other platforms are sometimes easier to fine-tune, and organisations with stringent performance requirements have been
squeezing every last byte and millisecond out of them for a long time now.

However, we have been defending JS as a language, and Node.js as an environment, from the early days of the company.
And we are still doing so!

Turns out that Node.js and [npm](https://www.npmjs.com/) can form the basis for a friendly toolkit,
and also be extensible and support efficient software.

In this post we share some of the realisations and tips that we have discovered and adopted along the way.

## Local config files are your friends

We rely extensively on [`.npmrc`](https://docs.npmjs.com/cli/v10/configuring-npm/npmrc)
and [`.nvmrc`](https://github.com/nvm-sh/nvm#nvmrc);
either locally (sitting at `$HOME` in each of our workstations), or
[shared with all other contributors and kept under version control](https://github.com/DevoInc/genesys-ui/blob/master/.nvmrc).

Depending on the nature of your project (proprietary or open source, on an internal repository or
[public on GitHub](https://github.com/orgs/DevoInc/repositories))
you will want your npm dependencies to be fetched from one source or another&nbsp;&mdash;&nbsp;and
the npm packages you produce to be published to the right place.
In fact, the consequences of using one endpoint instead of another might be dire (imagine publishing
your precious npm package `@corp/internal-trading-strategy` to `npmjs.com` by mistake).
Here's where an `.npmrc` file comes handy, as it lets you point to a specific set of npm repositories,
overriding global configuration, and even temporarily store your authentication credentials for
those services.

With regards to [nvm](https://github.com/nvm-sh/nvm), suffice to say that we maintain dozens of npm-based
projects with a wide range of requirements.
At any given point in time our engineers are working with several different versions of Node.js and npm,
so making sure that they always use the right one for each project saves time and prevents
mysterious build errors.

## Don't neglect pipelines and know when things are broken

What developer doesn't have the temptation to mute or simply ignore occasional failures of their
Continuous Integration or Continuous Deployment builds?
Sometimes overwhelmed by day-to-day work and more pressing issues, it's difficult to pay attention
to each and every failed build&nbsp;&mdash;&nbsp;not to mention to humble warnings.

However, we have found that striving to tend to those notifications pays in the long run.
Our front-end teams do a deliberate effort to optimise the number of automated checks and
to fine-tune the thresholds we set so that there are few false positives and errors are tackled.

It is not that time-consuming to spend ten minutes, every now and then, turning a few knobs and
toggling some checkboxes.
Let's see an example.

A pipeline job may be temporarily broken for &ldquo;good reasons&rdquo;; those include transient
misalignments between projects or dependencies, ongoing work on related pieces of software, and
downtime from third-party services (like when a linter or a scanner is under maintenance).
Most CI/CD tools provide syntax to keep on running a job that we accept may fail, so that the
result of the whole process isn't affected by that.
See eg [GitLab's CI `allow_failure` keyword](https://docs.gitlab.com/ee/ci/yaml/#allow_failure).

Preventing false positives in that way increases signal-to-noise ratio and contributes to
making actual problems more salient and therefore harder to ignore.

![A pipeline!](/assets/images/posts/nodejs-and-npm-best-practices/pipeline.webp)

## Don't be afraid to experiment

As you can see, npm and Node.js are at the root of our work with JavaScript.

In the last years we have also explored a few other platforms and dependency managers.
Among those,
[Deno](https://deno.com/),
[Yarn](https://yarnpkg.com/) or
[pnpm](https://pnpm.io/).
Recently, we had been excited about the promises (no pun intended) made by
[Bun](https://bun.sh/), and the possibility that it alone could replace (and improve)
a handful of items in our current tool chain.

Typically at Devo, some colleague conducts early experiments with a pet project of theirs
or with some non-critical component.
If their experience is positive, those results are usually shared with the front-end
architecture chapter, and in turn publicised or even recommended to all other coders.

Other excellent tools that we introduced following that playbook are
[StrykerJS](https://stryker-mutator.io/) for mutation testing,
[release-it](https://www.npmjs.com/package/release-it) to streamline versioning and
changelog editions, or
[Vite](https://vitejs.dev/) for building.
We switched from [Lerna](https://lerna.js.org/) to
[native monorepos with npm workspaces](https://docs.npmjs.com/cli/v10/using-npm/workspaces).
And of course, we have moved more and more of our codebase to
[TypeScript](https://www.typescriptlang.org/) following the strategy outlined above.

## &hellip;but don't chase every bright shiny object

Memes abound about the insane cycle of hype in JS-land.
We all know there is a kernel of truth to those funny images and blog posts: JavaScript is
a tremendously popular programming language (somewhere between
[#1](https://survey.stackoverflow.co/2023/#section-most-popular-technologies-programming-scripting-and-markup-languages)
and [#6](https://www.tiobe.com/tiobe-index/)) which makes it very easy for anyone to
develop and share anything from a tiny library to a gargantuan framework.
It's sometimes too easy to get excited about so many new techniques and thingies&hellip;
most of which will be unmaintained in a year's time.

At Devo, we try to strike a healthy balance between innovation and caution.
Having a diverse enough team of programmers helps here, because a wide range of ages,
backgrounds, interests and skill sets make for a better debate.
All the tools mentioned in the previous paragraph were the result of lively conversations
among engineers and of practical experimentation&nbsp;&mdash;&nbsp;not top-down
impositions or the whim of anyone in particular.

👉 If you or your team are suffering from &ldquo;JavaScript fatigue&rdquo;, Axel Rauschmayer has
[some useful tips](https://2ality.com/2016/02/js-fatigue-fatigue.html), such as
&ldquo;wait for the critical mass&rdquo; and &ldquo;don't use more than 1&ndash;2 new
technologies per project&rdquo;.

![Some bright shiny objects](/assets/images/posts/nodejs-and-npm-best-practices/bright-objects.jpeg)

## Streamline maintenance tasks

With large enough codebases, keeping things tidy, clean and updated is a task in itself,
something that could keep one person busy full time.
JavaScript projects are no exception: if unmaintained, they rot and decay quicker than
you can say &ldquo;unsupported engine&rdquo; or &ldquo;critical vulnerability&rdquo;.

For the purpose of this discussion, let's break down &ldquo;routine maintenance&rdquo; into
three different types of activities:

* _Creating_ stuff.  
  Examples: generate an artefact, publish a package, release a version.
* _Changing_ stuff.  
  Examples: update dependencies, fix broken links, amend documentation.
* _Deleting_ stuff.  
  Examples: remove unused dependencies, prune code that can't be reached,
  ditch config files that aren't useful any more.

The first type of maintenance is usually achieved through test suites, documentation comments,
programmatic generation of docs, and CI/CD.
In an ideal world, no human should ever have to take care of that directly: a new commit
being pushed, or the clock striking 3:00 AM, should trigger all changes necessary for the
latest version to be linted, tested, built, packaged and deployed _automagically_.

What fewer JS programmers know is that many of the maintenance tasks in the second category
(&ldquo;changing stuff&rdquo;) can be automated, too.

We are usually reluctant to let tools update npm dependencies on our behalf, and for good
reasons: the most innocent-looking change in our dependency tree could break everything,
and in principle there is no way to know.
So, how could we entrust that delicate task to an unattended process?
Enter packages
[`npm-check-updates` (in &ldquo;doctor mode&rdquo;)](https://www.npmjs.com/package/npm-check-updates#doctor)
and
[`updtr`](https://www.npmjs.com/package/updtr).
These two take care of upgrading all dependencies that can be upgraded,
within the semver range specified in `package.json`, and (most importantly)
are able to run an arbitrary npm script and use the result to decide whether
each individual upgrade is feasible or not.

For instance:

```shell
npx updtr --test "npm t && ./checks.sh && npm run whatever-you-usually-do"
```

_Voilà!_
Make that part or your scheduled pipeline, and see your dependencies always fresh
(assuming that your test suites and your checks are good enough, that is!).

👉 We mentioned broken links above.
For that, you could integrate the <abbr title="World Wide Web Consortium">W3C</abbr>'s
[Link Checker](https://github.com/w3c/link-checker) into your pipeline.

What about the third type of maintenance, &ldquo;deleting stuff&rdquo;?
For unused code, there's code coverage, and most teams use those metrics.
For unused npm dependencies, something like
[`npm-check`](https://www.npmjs.com/package/npm-check) comes to the rescue:

```shell
npx npm-check | grep -i 'notused?' | rev | cut -d'?' -f2 | cut -d' ' -f1 | rev
```

The one-liner above produces a list of packages that _probably_ aren't used by
your software any more.
Make sure to review manually though, as there could be false positives;
or, if you have enough confidence in your tests, automate the removal of
those dependencies _iff_ that change doesn't break the build.
