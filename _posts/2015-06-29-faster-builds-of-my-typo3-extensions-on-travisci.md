---
layout: post
title: Faster builds of my TYPO3 extensions on Travis CI
date: '2015-06-29T10:42:00.001+02:00'
author: Torben Hansen
tags:
- travis ci
- extension
- TYPO3
- build
- container based build
modified_time: '2015-06-29T10:44:44.029+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-9050159300648008165
blogger_orig_url: http://www.derhansen.de/2015/06/faster-builds-of-my-typo3-extensions-on-travisci.html
permalink: /2015/06/faster-builds-of-my-typo3-extensions-on-travisci.html
---

**tl;dr** If your Travis CI project was created before 01.01.2015 and you think your builds must run faster, switch to
container based builds by adding _sudo: false_ and configuring a cache for composer dependencies in your .travis.yml
file.

Nearly all my Open Source TYPO3 extensions are built on Travis CI. This runs really fine and I am very happy with the
service Travis CI offers for free if your project is Open Source. What I did'nt know was, that Travic CI also offers the
possibility to use container based builds (saw it first in
this [commit](https://github.com/TYPO3/TYPO3.CMS/commit/c758e6dd3c408e3b01bdc1efe94607f670a14271) for the 
**TYPO3 master)**. This feature
was [announced](http://blog.travis-ci.com/2014-12-17-faster-builds-with-container-based-infrastructure/) in the end of
2014 and has some advantages over the Travis CI standard infrastructure. Container based builds do have **more
resources** (2 cores vs. 1.5 cores and 4 GB Ram vs. 3 GB ram), **better network capacity** and can use a **cache** for
dependencies (e.g. composer dependencies).

My extension **sf\_event\_mgt** has a lot of functional tests and since I also collect code coverage data, builds did
run very long (15-30 minutes, worst result was more than 48 minutes) on the Travis CI standard infrastructure. Since my
Travis CI project was created before the **1st of january 2015**, my builds were automatically built using the old
infrastructure. I therefore tried to use the container based infrastructure to see if my builds would run faster.

In order to force your build to run on the container based infrastructure, you just have to add the following to your
.travis.yml file.

{% highlight yaml %}
sudo: false

cache:
  directories:
    - $HOME/.composer/cache
{% endhighlight %}

The _sudo: false_ option tells Travis CI to use the container based infrastructure and the cache configuration sets
the **cache-directory**, which will **persist during builds**.

When I started the first build, I was really amazed about how fast the builds started and how much faster builds were
completed. There was only one problem, which showed up in the logs.

{% highlight shell %}
  - Installing  symfony/yaml  ( v2.7.1 )
    Downloading:  connection...      Failed to download symfony/yaml from dist: Could not authenticate against github.com 
    Now trying to download from source 
  - Installing symfony/yaml  ( v2.7.1 )
    Cloning 9808e75c609a14f6db02f70fccf4ca4aab53c160
{% endhighlight %}

In order to fix the failing download of dependencies from github.com, you must create an OAUTH token for your GitHub
repository and add it to the .travis.yml. I therefore created an OAUTH token (which just has read access to my public
repositoried) in my GitHub account. Next I added the OATH token to the .travis.yml file. As the token should and must be
kept secret, Travis CI offers the possibility
to [encrypt sensitive data](http://docs.travis-ci.com/user/encryption-keys/). For my TYPO3 extension sf\_event\_mgt I
did as shown below:

{% highlight shell %}
sudo gem install travis
travis encrypt GITHUB_COMPOSER_AUTH=my-github-oauth-token -r derhansen/sf_event_mgt
{% endhighlight %}

Next I added the resulting encrypted key to my .travis.yml and configured composer to use the GitHub OAUTH token if
available.

{% highlight yaml %}
env:
  global:
    secure: my-encrypted-key

before_script:
  - if [ "$GITHUB_COMPOSER_AUTH" ]; then composer config -g github-oauth.github.com $GITHUB_COMPOSER_AUTH; fi
  - composer install
{% endhighlight %}

After restarting the build, the error did'nt show up in the logs of my Travis CI build any more. My complete
configuration can be found in this [gist](https://gist.github.com/derhansen/016c9e35647bef7bf486).

Finally, the switch to the container based builds seems to have **reduced the average build time** by **about 50%** and
seems to run **more stable** than before.