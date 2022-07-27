+++
title = "Using Zola (and other static site generators) with GitHub Pages without the need for an access token"
+++

## Moving from Jekyll to Zola

There are many static site generators:
[Jekyll](https://jekyllrb.com/),
[Hakyll](https://jaspervdj.be/hakyll/),
[Hugo](https://gohugo.io/),
[Hexo](https://hexo.io/),
[Lector](https://www.getlektor.com/),
[Sphinx](https://www.sphinx-doc.org/),
[MkDocs](https://www.mkdocs.org/),
[GitBook](https://www.gitbook.com/),
[Pelican](https://blog.getpelican.com/),
[11ty](https://www.11ty.dev/),
[Zola](https://www.getzola.org/),
and many others.

I have used Jekyll for many years in many projects and one of the reasons why
it is so popular is that it is supported by [GitHub
Pages](https://pages.github.com/) out of the box, without doing any extra
steps.  This makes it comparably simple to deploy blogs, homepages, and project
websites without running own web servers.

But recently I started exploring alternatives to Jekyll because it felt like
not a perfect fit for what I wanted to do. I often felt I am pressing something
else than a blog into a framework designed for blogs.  I was looking for
a framework where I can more easily customize things, where I achieve a better
separation between content and form, and where I know the underlying language
better.

In the recent weeks have tested many of the above tools and really liked Hugo
but my favorite static site generator at the moment is
[Zola](https://www.getzola.org/), and I have migrated already few projects to
Zola.  Zola really stood out, offers super fast build and rebuild, easy
installation, a link checker, clever image processing, shortcodes (reusable
macros), and much more. Overall the folder layout and the scaffold structure
maps better to how I think about a website.

What I wanted to find out is: **how can I use
Zola on GitHub Pages with the same ease as Jekyll: commit and push files and
that's it**.


## Deploying to GitHub Pages using GitHub Actions and using a personal access token

I was following
[this recipe](https://www.getzola.org/documentation/deployment/github-pages/#github-actions)
until now.

The approach described there requires creating a personal access token and
storing it as a *secret*.  This is not difficult and it only takes 2 minutes to
set up but I was never fully happy with this solution because of the
bus/lottery factor (how many persons have to be run over by a bus or win the
lottery for a project to become unmaintained). I wanted to create open source
websites which anybody can contribute to and which do not depend on me creating
and maintaining access tokens now or in future.


## How to build the page without creating a personal access token

It is indeed possible! And the approach described here should work for many of
the mentioned static site generators (I am using this not only for Zola but
also for pages built with Sphinx).

All you need to do is to add (and adapt, more about this later) the file
`.github/workflows/build.yml` to your GitHub repository with the following content:

```yml
name: Build website

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    # we use mac because zola is not distributed in ubuntu (?)
    # and I did not want to cargo build zola as part of the workflow
    runs-on: macos-10.15
    if: github.ref == 'refs/heads/main'
    env:
      TARGET_BRANCH: master
    steps:
    - name: Check out repo
      uses: actions/checkout@v2
    - name: Install zola
      run: brew install zola
    - name: Generate HTML
      run: zola build
    - name: Commit and push to target branch
      run: |-
        git config --global user.email "workflow-bot@example.com"
        git config --global user.name "workflow-bot"
        git checkout --orphan $TARGET_BRANCH
        git rm --cached -r .
        mv public ..
        rm -rf *
        mv ../public/* .
        touch .nojekyll
        git add .
        git commit -m "generated using zola build"
        git push --set-upstream origin $TARGET_BRANCH --force
```

You may need to adapt the source branch (this is where the Zola sources are; in
this example: `main`, mentioned three times) and the target branch (this is
the branch which contains the generated HTML and is used by GitHub Pages; in
this example: `TARGET_BRANCH: master`).

This workflow creates a new target branch every time the source branch is
modified.

- The target branch is `master` for homepages or project pages of the form
  `github.com/myuser/myuser.github.io` and
  `github.com/myproject/myproject.github.io`.

- For project pages of the form `github.com/mynamespace/myproject` the source
  branch is typically `main` and the target branch is typically `gh-pages`.

But the nice thing is that the above workflow is easily adaptable to either
situation.

One more question: why did I use `macos-10.15` for the build and not
`ubunutu-latest`? Because as of writing, surprisingly to me, Ubuntu does not
ship Zola ([many other distributions are
supported](https://www.getzola.org/documentation/getting-started/installation/)).
I did not want to build Zola from sources for every page rebuild (although it
would not have been difficult), so I went for a macOS build where I can install
Zola with a one-liner (`brew install zola`).

Once you adapt, commit, and push this file, all that remains is to enable
GitHub Pages under the repository settings page. That's it!


## Conclusion

[Zola](https://www.getzola.org/) is a really nice tool with great documentation - try it out!
Indeed, this website is built using Zola and
[here](https://github.com/bast/bast.github.io/blob/main/.github/workflows/build.yml)
is the workflow that I am using to build this site. *Edit: Do note the current workflow now relies on an Ubuntu builder which pulls the Linux Zola binaries directly from the official repository.*
With the above recipe setting up a Zola build is no more difficult than setting
up a Jekyll build.
