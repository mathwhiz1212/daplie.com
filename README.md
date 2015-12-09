Daplie.com
==========

<https://daplie.com> is built with [Desi](https://github.com/DearDesi/Desi),
the Static Site Generator for DIYers.

QuickStart
======

Install node.js and desi

```bash
echo "Current node.js version is $(curl -fsSL https://nodejs.org/dist/index.tab | head -2 | tail -1 | cut -f 1)"

curl -fsSL bit.ly/iojs-dev -o /tmp/iojs-dev.sh; bash /tmp/iojs-dev.sh

npm install -g desi
npm install -g serve-https
brew install fswatch
```

Build daplie.com

```bash
git clone git@github.com:Daplie/daplie.com.git

pushd

git submodule init
git submodule update

desi build
```

*TODO: fswatch*

In another terminal tab

```bash
pushd daplie.com/compiled

sudo serve-https 443 80 --livereload
```

TODO: fswatch, live reload

How we Got Here
=====

1. Install node.js
----------

```bash
echo "Current node.js version is $(curl -fsSL https://nodejs.org/dist/index.tab | head -2 | tail -1 | cut -f 1)"

curl -fsSL bit.ly/iojs-dev -o /tmp/iojs-dev.sh; bash /tmp/iojs-dev.sh
```

2. Install Desi
----------

```bash
npm install -g desi
```

3. Create a new Site
----------

```bash
# initialize a new blog directory
desi init -d ~/daplie.com

# push into the new directory
pushd ~/daplie.com

# see the layout, ignoring the themes folder
tree -I themes
```

4. Configure
----------

You can use the Web interface:

```bash
desi serve -d ~/daplie.com
```

Or edit via the command line

```
site.yml
config.yml
authors/<<your-twitter-handle>>.yml
```

<http://localhost:65080>

5. Initial Commit
----------

```bash
git init
git remote add origin git@github.com:Daplie/daplie.com.git

git add ./
git reset themes LICENSE README.md

git commit -m "initial commit"
```

Alternatively

```bash
git add \
  ./partials.yml \
  ./config.yml \
  ./site.yml \
  ./authors \
  ./posts \
  ./_root/ \
  ./themes/.gitkeep \
  ./media/.gitkeep \
  ./.gitignore
```

5. Your First Post
------------------

You can use the web interface

```bash
desi serve -d ~/daplie.com
```

Or use the desi cli

```bash
desi post "My First Post"
```

This will create

```
posts/my-first-post.md
```

Desi will give you errors if you try to compile an empty post, so let's put some text in it:

```
echo "Hello World!" >> posts/my-first-post.md
```

5. Add a Theme
----------

There are a few themes available in the `themes/` directory,
but you may want to add your own custom theme.

```bash
git submodule add -f git@github.com:Daplie/desi-theme-daplie.git themes/daplie
```

* Add the theme to the list of available themes in `config.yml`

`config.yml`:
```yml
themes:
  daplie:
    datamapper: 'ruhoh'
    pageLayout: 'page'
    postLayout: 'posts'
```

* Set your theme to be the default for the site

`site.yml`:
```yml
theme: daplie
```

* Set different themes for different posts

By defaut the site theme will be used.
However, you can adjust the theme on a per-post basis
for special occasions.

```bash
desi post "Holiday-themed Post!"

echo 'Merry Christmas!' >> posts/holiday-themed-post.md
```

`posts/holiday-themed-post.md`
```yml
---
title: 'Holiday Special!'
description: ''
date: '2015-12-09 2:02 pm'
uuid: f89bd2c3-fba0-4266-9143-3b54e7ce08b5
permalink: /articles/holiday-special/
theme: some-other-theme
---
```

Note

```
theme: some-other-theme
```

## BUGS

* CLI needs setup wizard
* `desi post` requires `site.yml` to have `base_path`.
* HSTS http -> https affects localhost
* themes section in config.yml should be automated from the theme manifest.json or manifest.yml
* needs better error message for site.yml bad theme and themes listed in config.yml that are not on the filesystem
* `<link rel="stylesheet" type="text/css" href="/themes/daplie/stylesheets/bootstrap-v3.3.5.min.css">`
* error message for when theme is missing from config.yml (pageLayout undefined)
* copy fonts directory
