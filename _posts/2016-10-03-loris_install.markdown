---
layout: post
title:  "Loris Installation"
date:   2016-10-03 10:43:00
categories: iiif loris
---

Although I have successfully added the [RIIIF][riiif] gem to the DRI application, it occurred to me
that I should try out [Loris][loris] as an alternative image server. First step is to install it
on one of our Ubuntu VM 14.04 hosts. Here are the steps I followed, which are mostly a slightly updated
version of those given [here][loris-install].

Install the dependencies:

{% highlight sh %}
$ sudo apt-get install libjpeg-turbo8-dev libfreetype6-dev zlib1g-dev
  liblcms2-dev liblcms-utils libtiff5-dev python-dev python-imaging
  libwebp-dev apache2 libapache2-mod-wsgi
{% endhighlight %}

{% highlight sh %}
$ apt-get install python-pip
$ pip install Werkzeug
{% endhighlight %}

Install Kakadu:

{% highlight sh %}
$ wget http://kakadusoftware.com/wp-content/uploads/2014/06/KDU78_Demo_Apps_for_Linux-x86-64_160226.zip
$ apt-get install unzip
$ unzip KDU78_Demo_Apps_for_Linux-x86-64_160226.zip
$ cp KDU78_Demo_Apps_for_Linux-x86-64_160226/libkdu_v78R.so \
   /usr/local/lib/
$ cp KDU78_Demo_Apps_for_Linux-x86-64_160226/kdu_expand \
   /usr/local/bin
{% endhighlight %}

Check Kakadu installation:

{% highlight sh %}
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
$ /usr/local/bin/kdu_expand -v
This is Kakadu's "kdu_expand" application.
    Compiled against the Kakadu core system, version v7.8
    Current core system version is v7.8
{% endhighlight %}

Now get the Loris source code from Github, add a Loris user and run the setup script.

{% highlight sh %}
$ git clone https://github.com/loris-imageserver/loris
$ cd loris
$ useradd -d /var/www/loris2 -s /sbin/false loris
$ ./setup.py install
{% endhighlight %}

Deploy with Apache:

{% highlight sh %}
$ a2enmod headers expires
$ a2ensite 000-default.conf
{% endhighlight %}

Add the following to the default Apache site (/etc/apache2/sites-enabled/000-default.conf):
{% highlight sh %}
ExpiresActive On
ExpiresDefault "access plus 5184000 seconds"

AllowEncodedSlashes On

WSGIDaemonProcess loris2 user=loris group=loris processes=10 threads=15 maximum-requests=10000
WSGIScriptAlias /loris /var/www/loris2/loris2.wsgi
WSGIProcessGroup loris2
{% endhighlight %}

After starting Apache you should be able to browse to http://\<host\>/loris.

To test that everything is working OK, copy the test images to the image source root:

{% highlight sh %}
$ mkdir /usr/local/share/images
$ cp -R tests/img/* /usr/local/share/images/
{% endhighlight %}

You should now be able to browse to the following URLs:

http://\<host\>/loris/01/02/0001.jp2/full/full/0/default.jpg
http://\<host\>/loris/01/03/0001.jpg/full/full/0/default.jpg
http://\<host\>/loris/01/04/0001.tif/full/full/0/default.jpg

[iiif]:              http://iiif.io/
[riiif]:             https://github.com/curationexperts/riiif
[loris]:             https://github.com/loris-imageserver/loris
[loris-install]:     http://doc.biblissima-condorcet.fr/loris-setup-guide-ubuntu-debian
