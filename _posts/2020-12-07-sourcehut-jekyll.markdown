---
layout: post
title:  "Tutorial: Deploying a Jekyll blog using sourcehut's builds.sr.ht"
date:   2020-12-07 10:22:24 -0600
categories: jekyll sourcehut build.sr.ht
---
### Introduction.

This tutorial aims at helping you deploy a Jekyll blog inside a git repo to a GNU/Linux server with the help of <a href="https://builds.sr.ht/" target="_blank">builds.sr.ht</a>.

#### Prerequisites

1. GNU/Linux Server - hosts your blog
2. Jekyll blog `git` repo - on any git server provider; Sourcehut, Github, Gitlab etc.
3. A <a href="https://builds.sr.ht/" target="_blank">builds.sr.ht</a> account.

### Defining the build manifest

Much like `travis.yml` or `circle.yml` files, build manifests define how <a href="https://builds.sr.ht/" target="_blank">builds.sr.ht</a> will build, test and deploy your application. A quick rundown on build manifests can be found here https://man.sr.ht/builds.sr.ht/#build-manifests. The relevant build manifest is shown below.

{% highlight yaml %}
image: archlinux
sources:
  - 'git@github.com:49e94b8f256530dc0d41f740dfe8a4c1/blog.git'
secrets:
  - 9f4e9cb9-f642-427b-96e1-c6fc6a5781f8
  - b03d783e-d793-479d-8558-082abb0ab74a 
  - 7619d574-cfe1-4a08-9a1d-df3a5499c31e
packages:
  - jekyll
  - rubygems
tasks:
  - setup: |
      jekyll -v
      gem install bundler
  - build: |
      cd blog
      echo $GEM_HOME
      PATH=$PATH:$HOME/.gem/ruby/2.7.0/bin
      bundle install
      bundle exec jekyll build
  - deploy: |
      cd blog
      DEPLOY_HOST=168.119.234.216
      DEPLOY_USER=ken
      eval `ssh-agent`
      ssh-add ~/.ssh/b03d783e-d793-479d-8558-082abb0ab74a
      ln -s ~/.ssh/7619d574-cfe1-4a08-9a1d-df3a5499c31e ~/.ssh/id_rsa.pub
      scp -o StrictHostKeyChecking=no -rv _site/* $DEPLOY_USER@$DEPLOY_HOST:/var/www/html/blog
{% endhighlight %}

{% highlight bash %}
$ jekyll new .
{% endhighlight %}

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
