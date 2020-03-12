---
title: "Yet Another Rails App in Subdirectory Howto"
layout: default
date: 2020-03-12 11:33:39 GMT
categories: rails deployment
---

Last week I tried, unsuccessfully, to help someone on Slack with deploying a Rails application in a 
subdirectory. By which I mean something like http://example.com/subdir instead of just the root as is
more usually the case. As it turns out I then needed to do this myself this week and had just as many
problems. As least some of the difficulty comes from there seeming to be a few different ways to achieve
this, as a Google search will quickly show. So what the internet definitely doesn't need is yet another
howto. So here is one. To try and at least prevent some confusion this is for a Rails 5 application
deployed using Apache and Passenger.

After a few attempts with various environment variables the simplest working solution was to use
```PassengerBaseURI``` in the Apache site configuration.

```
<VirtualHost *:80>
   ServerName https://example.com
   DocumentRoot /var/www/myapp

   <Directory /var/www>
      Options -MultiViews
      PassengerBaseURI /subdir
      PassengerAppRoot /websites/myapp
   </Directory>

</VirtualHost>
```

Here the document root, ```/var/www/myapp``` is a symbolic link to the PassengerAppRoot. 

The benefit with this is that there seems to be very few changes needed in the application itself. 
The only real modification is to do with the assets. As the application will be running in the production 
environment the assets will be served by Apache, not the Rails application. Apache will try to 
find the assets at ```public/subdir/assets```, so we need to tell Rails to generate them there 
rather than the default location. This can be done with the ```config.assets.prefix``` configuration setting. 
Add this to ```config/environments/production.rb```

```
config.assets.prefix = "/subdir/assets"
```

Now when the assets are precompiled they will be created in the correct location:

```
RAILS_ENV=production bundle exec rake assets:precompile
```

That should be all that is required to be able to access the application at the subdirectory as required.

