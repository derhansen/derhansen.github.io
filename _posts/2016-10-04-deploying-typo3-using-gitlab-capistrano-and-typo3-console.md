---
layout: post
title: Testing and deploying TYPO3 using GitLab, GitLab CI, Capistrano and TYPO3 Console
date: '2016-10-04T09:32:00.000+02:00'
author: Torben Hansen
tags:
- Capistrano
- GitLab
- Continuous Deployment
- GitLab CI
- TYPO3 Console
- TYPO3
- continuous integration
modified_time: '2016-10-04T09:32:22.824+02:00'
thumbnail: https://2.bp.blogspot.com/-sv9mpm9mAN4/V-0kNhO69mI/AAAAAAAASb8/hOkbhub9SyAByP4LYERDTcAeZwa1I9M7gCLcB/s72-c/Bildschirmfoto%2B2016-09-29%2Bum%2B16.23.09.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-4114996449039767109
blogger_orig_url: http://www.derhansen.de/2016/10/deploying-typo3-using-gitlab-capistrano-and-typo3-console.html
permalink: /2016/10/deploying-typo3-using-gitlab-capistrano-and-typo3-console.html
---

In this article, I'll show you how to setup automated testing and continuous deployment of a TYPO3 website
using **[GitLab](https://gitlab.com/gitlab-org/gitlab-ce)**
, **[GitLab CI](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner)**, **[Capistrano](http://capistranorb.com/)**
and **[TYPO3 Console](https://github.com/helhum/typo3_console)**. The main purpose of the article is to give you a _
starting point_ for automated testing and continuous deployment.

With the setup described in this article, I want to make sure, that

* tests are executed automatically on every commit
* pushes to master branch will automatically be deployed to the production server
* deployment will only execute, when tests pass

I tried to keep write the article as simple as possible, so please note, that some features / processes of real life
automated deployment scenarios are not covered.

**Prerequisites:**

* A **composer based TYPO3** website with a basic site package
* At least one TYPO3 extension with tests
* A working GitLab server (I'm using an own instance of the Community Edition)
* A GitLab CI runner
* A SSH account on the webspace, where you want the TYPO3 website to be deployed
* Ruby version 1.9.3 or newer on your local machine (required for capistrano)

[GitLab.com](http://gitlab.com/) offers _free private repositories_ and also access to GitLab CI runners, so I'm pretty
sure, that the setup I described here may even work using the GitLab.com infrastructure.

### 1\. Structure of the composer based TYPO3 website

The TYPO3 website used for this blogpost is a really simple composer based TYPO3 website. It contains a sitepackage with
TypoScript and a Fluid template and it contains the TYPO3
extension [sf\_yubikey](https://typo3.org/extensions/repository/view/sf_yubikey), which is used in this example to
execute some unit tests. Also the TYPO3 website contains the TYPO3
extension [typo3\_console](https://github.com/TYPO3-Console/typo3_console), which will be used in the deployment
process.

Unit tests can be executed on CLI with the command shown below:

{% highlight shell %}
./vendor/bin/phpunit -c web/typo3conf/ext/sf_yubikey/Tests/Build/UnitTests.xml
{% endhighlight %}

My composer.json file contains the following contents:

{% highlight json %}
{
 "repositories": [
  { "type": "composer", "url": "https://composer.typo3.org/" }
 ],
 "name": "typo3/cms-base-distribution",
 "description" : "TYPO3 CMS Base Distribution",
 "license": "GPL-2.0+",
 "require": {
  "typo3/cms": "^7.6",
  "typo3-ter/sf-yubikey": "*",
  "helhum/typo3-console": "^3.3"
 },
 "require-dev": {
   "mikey179/vfsStream": "1.4.*@dev",
   "phpunit/phpunit": "~4.7.0"
 },
 "extra": {
  "typo3/cms": {
   "cms-package-dir": "{$vendor-dir}/typo3/cms",
   "web-dir": "web"
  },
  "helhum/typo3-console": {
   "install-binary": false
  }
 }
}
{% endhighlight %}

Note, that I use **helhum/typo3-console** (not from TER), which on install makes the typo3cms command available in the _
vendor/bin_ directory.

The TYPO3 installation also contains a backend user named **\_cli\_lowlevel**, which is required to execute CLI
commands.

### 2\. Move all sensitive data and local settings out of the LocalConfiguration.php

You should never commit any sensitive data (like passwords or API keys) to a Git repository, so I move those settings
out of from LocalConfiguration.php to the file AdditionalConfiguration.php, which also is located in the typo3conf/
folder.

The local AdditionalConfiguration.php file used in this blogpost looks like shown below:

{% highlight php %}
<?php
$GLOBALS['TYPO3_CONF_VARS']['DB']['database'] = 'typo3db';
$GLOBALS['TYPO3_CONF_VARS']['DB']['host']     = 'localhost';
$GLOBALS['TYPO3_CONF_VARS']['DB']['username'] = 'dbusername';
$GLOBALS['TYPO3_CONF_VARS']['DB']['password'] = 'dbpassword';

$GLOBALS['TYPO3_CONF_VARS']['BE']['installToolPassword'] = 'hashvalue';

$GLOBALS['TYPO3_CONF_VARS']['GFX']['im_path'] = '/opt/local/bin/';
$GLOBALS['TYPO3_CONF_VARS']['GFX']['im_path_lzw'] = '/opt/local/bin/';
{% endhighlight %}

Besides all sensitive data from the LocalConfiguration.php, the AdditionalConfiguration.php file should include all
settings, that should not be shared across deployment stages.

### 3\. Modify the default .gitignore, so some TYPO3 directories will not be included in the git repository

For the TYPO3 website in this blogpost, I extended the default .gitignore file, so contents of some TYPO3 directories
and files do not get committed to the git repository (e.g. fileadmin, uploads or all extensions installed by composer)

{% highlight text %}
#############################
# TYPO3 CMS Base Distribution
#############################
.DS_Store
.idea
index.php
nbproject
node_modules
vendor
typo3
typo3_src
web/index.php
web/typo3

# Ignore fileadmin, uploads and typo3temp
web/fileadmin
web/uploads
web/typo3temp

# Ignore some files in typo3conf
web/typo3conf/ENABLE_INSTALL_TOOL
web/typo3conf/AdditionalConfiguration.php
web/typo3conf/*.log

# Ignore language files (fetched by TYPO3 console)
web/typo3conf/l10n

# Ignore all extensions (loaded by composer)
web/typo3conf/ext/*

# But include the sitepackage, which is not composer based
!web/typo3conf/ext/sitepackage

# Ignore capistrano deployment logs
log
{% endhighlight %}

### **4\. Commit the TYPO3 website to a git repository**

The .gitignore file is ready, so now I create a new git repository and add / commit all files to it. Finally, I create a
new project on the **GitLab server** and add the repository as a remote for the local git repository and finally push
the repository to the remote.

{% highlight shell %}
git remote add origin git@gitlab.com:derhansen/typo3_ci_cd.git
git push -u origin master
{% endhighlight %}

### **5\. Configure deployment of TYPO3 website with capistrano**

5.1 SSH setup for deployment to webserver

First, I add my public SSH RSA key to the .**ssh/authorized\_keys** file in the home-directory of the SSH user on the
webserver, so I'm able to login by using SSH key authentication. This step is not required, but makes deployment from my
local machine to the webserver easier.

Next I create a **new SSH RSA key** on the webserver (like
shown [here](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)), so the SSH
user will be able to pull from the remote git repository on the GitLab server I created in step 4. This SSH RSA Key will
be the **deployment key** and must not contain a password.

The new **SSH public key** must now be added to the git repository on the GitLab server. In order to do so, I add it as
Deployment Key for the project (see screenshot below)

![](/assets/images/2016-10-04/image1.png)

5.2 Initialize and configure capistrano

I have chosen capistrano as the deployment tool for the TYPO3 website, because it is well documented, has a large user
base and is easy to use/extend. If you do not like capistrano, it should be easy to replace capistrano with the
deployment tool of your choice (e.g. TYPO3 surf, Deployer, Magallanes, ...)

In the project root of my local composer based TYPO3 website, I now initialize a new capistrano config with the
following command:

{% highlight shell %}
cap install
{% endhighlight %}

First of all, I remove everything in the file **config/deploy/production.rb** and add the following content:

{% highlight yaml %}
server 'typo3-ci-cd.domain.tld',
  user: 'typo3cicd',
  roles: %w{app db web}
{% endhighlight %}

Next, I create a really basic deployment in **config/deploy.rb**, which mainly executes composer install and executes
some TYPO3 tasks using TYPO3 console.

{% highlight ruby %}
# config valid only for current version of Capistrano
lock '3.6.1'

set :application, 'typo3_ci_cd'
set :repo_url, 'git@path-to-my-gitlab:torben/typo3_ci_cd.git'
set :deploy_to, '/var/www/typo3-ci-cd/httpdocs'
set :scm, :git
set :keep_releases, 5
set :linked_dirs, %w{web/fileadmin web/uploads}
set :linked_files, %w{web/typo3conf/AdditionalConfiguration.php}

namespace :composer do
    desc "Runs composer."
    task :install do
        on roles(:web) do
            within release_path do
                execute "composer", "install", "--no-dev", "--no-interaction", "--prefer-dist"
            end
        end
    end
end

namespace :typo3 do

    desc "Clear TYPO3 Cache"
    task :cache_flush do
        on roles(:all) do
            within release_path do
                execute "#{release_path}/vendor/bin/typo3cms cache:flush"
            end
        end
    end

    desc "Update language"
    task :language_update do
        on roles(:all) do
            within release_path do
                execute "#{release_path}/vendor/bin/typo3cms language:update"
            end
        end
    end

end

namespace :deploy do
    after :publishing, "composer:install"
    after :finishing,  "typo3:cache_flush"
    after :finishing,  "typo3:language_update"
end
{% endhighlight %}

You may note the file AdditionalConfiguration.php in the **linked\_files** section of the deployment configuration. This
is required in order to get TYPO3 running, when I'm doing a deployment from my local machine.

The **linked\_dirs** section contains the folders **web/fileadmin** and **web/uploads**, which I want to share across
deployments, since users will upload files to it.

5.3 Manual steps before first deployment

Before the first deployment can be executed, there are some steps, that need to be done manually on the webserver.

_5.3.1 Initial shared folder setup_

I copy the contents of the directories **fileadmin/** and **uploads/** from my local TYPO3 website to the directory **
shared/web** in the deployment path (see "deploy\_to" in the deploy.rb file). The final directory structure is as shown
below (Note: .htaccess files are not shown).

{% highlight text %}
shared/
└── web
    ├── fileadmin
    │   ├── _temp_
    │   │   └── index.html
    │   └── user_upload
    │       ├── index.html
    │       └── _temp_
    │           ├── importexport
    │           │   └── index.html
    │           └── index.html
    ├── typo3conf
    │   └── AdditionalConfiguration.php
    └── uploads
        ├── index.html
        ├── media
        │   └── index.html
        ├── pics
        │   └── index.html
        └── tx_felogin
{% endhighlight %}

The shared folder does also contains the **typo3conf/** directory, where I add the **AdditionalConfiguration.php** and
modify the settings (e.g. database credentials, GraphicsMagick path, ...) to match the server configuration.

_5.3.2 Initial database setup_

Since there is no initial TYPO3 database on the webserver, I create a local dump of the TYPO3 database and restore it to
the remote database on the webserver.

_5.4 Initial deployment_

Now it is time to do the first deployment form my local machine, so I execute the deployment with the following command

{% highlight shell %}
cap production deploy
{% endhighlight %}

Capistrano now executes the deployment and after finishing, I have a fully working copy of my local TYPO3 website on the
remote webserver.

Capistrano creates the directories **current/**, **releases/**, **repo/** and **shared/.** The current version of the
deployed TYPO3 website is available in the current/ directory, so I adjusted the webserver to load the website from that
directory.

_5.5 Commit deployment configuration_

Finally I commit the deployment configuration to the git repository, so it later can be used for the automated
deployment.

### **6\. Configure automated test execution on GitLab server**

In an automated deployment process, you need to make sure, that only working versions of your website will be deployed
to the production server. In this example, I simply run the unit tests of a TYPO3 extension to demonstrate, how
automatic test execution can be a part of a deployment process. In real life scenarios, you can for example use unit,
functional or acceptance tests to make sure, that your website will run smoothly after deployment.

In my local project, I create the file **.gitlab-ci.yml** and add the following contents to it:

{% highlight yaml %}
stages:
  - tests

phpunit:php7.0:
  stage: tests
  image: php:7.0
  before_script:
    - apt-get update -yqq
    - apt-get install git libcurl4-gnutls-dev libicu-dev libvpx-dev libjpeg-dev libpng-dev libxpm-dev zlib1g-dev libfreetype6-dev libxml2-dev libexpat1-dev libbz2-dev libgmp3-dev libldap2-dev unixodbc-dev libpq-dev libsqlite3-dev libaspell-dev libsnmp-dev libpcre3-dev libtidy-dev -yqq
    - docker-php-ext-install mbstring zip
    - curl -sS https://getcomposer.org/installer | php
  script:
    - php composer.phar install
    - php vendor/bin/phpunit --colors -c web/typo3conf/ext/sf_yubikey/Tests/Build/UnitTests.xml
{% endhighlight %}

I add and commit the file to the git repository and push the changes to the remote. Now GitLab CI will start a **PHP
7.0** **docker container**, install all dependencies and finally run the configured unit tests.

Nice, the build passed....

![](/assets/images/2016-10-04/image2.png)

...and unit tests have been executed on the GitLab CI runner.

![](/assets/images/2016-10-04/image3.png)

You may notice, that the build took _3:16 minutes_ to finish, which is pretty long just for unit test execution. To
reduce the build time, I suggest
to [create your own docker image](https://docs.docker.com/engine/tutorials/dockerimages/#/building-an-image-from-a-dockerfile)
with all dependencies included.

If you want to **receive e-mail notifications** on builds (or only for failed builds), you can enable this feature in
the [service section](https://docs.gitlab.com/ce/project_services/builds_emails.html) of the GitLab project.

### **7\. Configure automated deployment on GitLab server**

As a final step, I want to automate the deployment of the website, when commits are made to the master branch and if
tests passed. Since the GitLab CI runner will do the deployment, it must be able to SSH to the production website.

I create a new SSH RSA key (with no password), which will be used for the deployment from the GitLab CI runner to the
production webserver. After adding the **SSH public key** to the **ssh/authorized\_keys** file of the SSH user on the
webserver, I create a new variable in the GitLab project and add the new private key to it.

![](/assets/images/2016-10-04/image4.png)

Next, I extend the **.gitlab-ci.yml** to use the **SSH\_PRIVATE\_KEY** variable in the docker container, that will do
the deployment. Detailed instructions can be found in
the [GitLab CI documentation](https://docs.gitlab.com/ce/ci/ssh_keys/README.html). The final configuration file looks
like shown below:

{% highlight yaml %}
stages:
  - test
  - deploy

phpunit:php7.0:
  stage: test
  image: php:7.0
  before_script:
    - apt-get update -yqq
    - apt-get install git libcurl4-gnutls-dev libicu-dev libvpx-dev libjpeg-dev libpng-dev libxpm-dev zlib1g-dev libfreetype6-dev libxml2-dev libexpat1-dev libbz2-dev libgmp3-dev libldap2-dev unixodbc-dev libpq-dev libsqlite3-dev libaspell-dev libsnmp-dev libpcre3-dev libtidy-dev -yqq
    - docker-php-ext-install mbstring zip
    - curl -sS https://getcomposer.org/installer | php
  script:
    - php composer.phar install
    - php vendor/bin/phpunit --colors -c web/typo3conf/ext/sf_yubikey/Tests/Build/UnitTests.xml

deployment:
  stage: deploy
  image: ruby:2.3
  before_script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SSH_PRIVATE_KEY")
    - mkdir -p ~/.ssh
    - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
  script:
    - gem install capistrano
    - cap production deploy
  only:
    - master
{% endhighlight %}

Since capistrano is a ruby application, I use a **docker container** **with ruby** for the deployment.

Finally, after committing the changes to the repository, the GitLab CI runner runs the build and deploys the TYPO3
website to the production server.

![](/assets/images/2016-10-04/image5.png)

And that's it, the automated testing and continuous deployment setup for TYPO3 is up and running.

### Outlook and conclusion

There are for sure some things that can be improved in the process I described in this post. For example, you could also
add the AdditionalConfiguration.php file as a secret variable to the GitLab project and use the GitLab CI Mutli-Runner
to deploy it to the production server. Also you can automate TYPO3 related tasks using TYPO3 console (e.g. database
updates on new extension installation with _typo3cms database:updateschema_, update reference index, ...).

I hope the post will give you a good point of how/where to start with automating tests and continuous deployment of a
TYPO3 website. The shown technique is _just one way_ of how to setup and automate the process - feel free to build your
own configuration with the tools of your choice and please share your knowledge :-)