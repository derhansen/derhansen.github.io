---
layout: post
title: Upgrading to Redmine 3.0.1 on Debian Wheezy
date: '2015-03-31T11:09:00.000+02:00'
author: Torben Hansen
tags:
- Debian Wheezy
- phusion passenger
- redmine 3.0
modified_time: '2015-03-31T11:09:01.534+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-6230429445931720064
blogger_orig_url: http://www.derhansen.de/2015/03/upgrading-to-redmine-301-on-debian.html
permalink: /2015/03/upgrading-to-redmine-301-on-debian.html
---

Today I wanted to upgrade my Redmine installation from **Redmine 2.6.x** to the new version **Redmine 3.0.1**. Since I
already have processed many Redmine updates I thought this update would be easy as always, but certainly I ran into an
unexpected problem. After I followed all steps from
the [upgrade manual](http://www.redmine.org/projects/redmine/wiki/RedmineUpgrade) I restarted the apache2 process, so **
mod\_passenger** reloads the Redmine application. After that, Redmine did'nt start and showed the following error
message:

{% highlight text %}
undefined method `page_cache_directory' for ActionController::Base:Class
{% endhighlight %}

After doing some Google search, I found
this [article](http://stackoverflow.com/questions/21531485/phusionpassengerclassicrailsapplicationspawner-undefined-method-page-cache)
on stackoverflow, which pointed me to the right direction. Actually, Debian Wheezy comes with an pretty **old version of
Phusion Passenger (3.0.x)**, so I performed the following steps and installed a newer version (4.0.x) of Phusion
Passenger from Wheezy Backports.

First I added Wheezy Backports to my apt sources file /etc/apt/sources.list

{% highlight shell %}
deb http://http.debian.net/debian wheezy-backports main
{% endhighlight %}

Next I used **apt-get update** to update the local apt repositories and finally I installed Phusion Passenger from the
Wheezy Backports with the following command

{% highlight shell %}
apt-get -t wheezy-backports install "libapache2-mod-passenger"
{% endhighlight %}

After this, I restarted the apache2 process and Redmine 3.0.1 started successfully.