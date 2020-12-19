---
layout: post
title: "Deploying a Jekyll blog with sourcehut"
description: "This tutorial shows how to deploy a Jekyll blog to a GNU/Linux server with the help of Sourcehut"
date: 2020-12-07 10:22:24 -0600
tags: [ci, jekyll, sourcehut, builds.sr.ht]
---

This tutorial shows how to deploy a Jekyll blog to a GNU/Linux server with the help of [builds.sr.ht][builds.sr.ht]{:target="\_blank"}

#### Prerequisites

1. GNU/Linux Server - hosts your blog
2. Jekyll blog - hosted on any `git` server provider; Sourcehut, GitHub, GitLab etc.
3. A [builds.sr.ht][builds.sr.ht]{:target="\_blank"} account.

### Defining the build manifest

Much like `travis.yml` or `circle.yml` files, [manifests][man.builds.sr.ht-build-manifests]{:target="\_blank"} describe a build. The relevant build manifest is shown below.

{% highlight yaml %}
image: archlinux
sources:

- 'git@github.com:49e94b8f256530dc0d41f740dfe8a4c1/blog.git'
  secrets:
- <secret-uuid-1>
- <secret-uuid-2>
- <secret-uuid-3>
  packages:
- jekyll
- rubygems
  tasks:
- setup: |
  jekyll -v
  gem install bundler
- build: |
  cd blog
  GEM_DIR="$(ruby -e 'puts Gem.user_dir')"
  $GEM_DIR/bin/bundle install
  $GEM_DIR/bin/bundle exec jekyll build
- deploy: |
  cd blog
  DEPLOY_HOST=<host>
  DEPLOY_USER=<user>
  eval `ssh-agent`
  ssh-add ~/.ssh/b03d783e-d793-479d-8558-082abb0ab74a
  ln -s ~/.ssh/7619d574-cfe1-4a08-9a1d-df3a5499c31e ~/.ssh/id_rsa.pub
  scp -o StrictHostKeyChecking=no -rv \_site/\* $DEPLOY_USER@$DEPLOY_HOST:/var/www/html/blog
  {% endhighlight %}

Although a general description of the build manifest schema can be found [here][man.builds.sr.ht-build-manifests]{:target="\_blank"}, I'll continue by detailing the Jekyll specific configuration options below.

#### Image

[builds.sr.ht][builds.sr.ht]{:target="\_blank"} offers a ton of images to choose from. `archlinux` was chosen because of its relatively superior documentation and near vanilla/as close to possible to upstream packages.

{% highlight yaml %}
image: archlinux
{% endhighlight %}

#### Sources

Specify what repo to clone. The `ssh` schema is used if the repo in question is private, otherwise use the `https` schema.

{% highlight yaml %}
sources:

- 'git@github.com:<username>/<repo>.git'
  {% endhighlight %}

#### Secrets

A YAML list of `uuid`s that point to secrets specified [here][builds.sr.ht-secrets]{:target="\_blank"}. Secrets can be SSH, PGP keys or a file.

{% highlight yaml %}
secrets:

- <secret-uuid-1>
- <secret-uuid-2>
- <secret-uuid-3>
  {% endhighlight %}

For this build however, we are only interested in SSH secrets. We need the deploy server's public and private key.

> NOTE:
> 3 secret uuids are needed because the git repo is private.
>
> The private key needed for `git clone` therefore occupies the first slot.
>
> However, public repos just need 2 uuids i.e. the deploy server key pair.

The first key on the list is linked to `~/.ssh/id_rsa`. Subsequent keys can be added using `ssh-add`.

Generate a key pair with `ssh-keygen`. Do not use a passphrase when prompted.

{% highlight bash %}
ssh-keygen -t rsa
{% endhighlight %}

Copy the public key over to the deploy server with `ssh-copy-id`

{% highlight bash %}
ssh-copy-id -i ~/.ssh/id_rsa user@host
{% endhighlight %}

#### Packages

Install any packages needed as dependencies.

{% highlight yaml %}
packages:

- jekyll
- rubygems
  {% endhighlight %}

This is the equivalent of

{% highlight bashl %}
sudo pacman -S jekyll rubygems
{% endhighlight %}

#### Tasks

YAML list of command line instructions needed to successfully run the build.

{% highlight yaml %}
tasks:

- setup: |
  jekyll -v
  gem install bundler
- build: |
  cd blog
  GEM_DIR="$(ruby -e 'puts Gem.user_dir')"
  $GEM_DIR/bin/bundle install
  $GEM_DIR/bin/bundle exec jekyll build
- deploy: |
  cd blog
  DEPLOY_HOST=<host>
  DEPLOY_USER=<user>
  eval `ssh-agent`
  ssh-add ~/.ssh/<secret-uuid-2>
  ln -s ~/.ssh/<secret-uuid-3> ~/.ssh/id_rsa.pub
  scp -o StrictHostKeyChecking=no -rv \_site/\* $DEPLOY_USER@$DEPLOY_HOST:/var/www/html/blog
  {% endhighlight %}

[builds.sr.ht]: https://builds.sr.ht
[builds.sr.ht-secrets]: https://builds.sr.ht/secrets
[man.builds.sr.ht-supported-images]: https://man.sr.ht/builds.sr.ht/compatibility.md
[man.builds.sr.ht-build-manifests]: https://man.sr.ht/builds.sr.ht/#build-manifests
[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
