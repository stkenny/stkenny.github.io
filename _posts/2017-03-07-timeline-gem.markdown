---
layout: post
title:  "I've Been Working on the Timeline..."
date:   2017-03-07 18:59:00
categories: rails javascript
---

One of the features added to the DRI repository just before launch was a timeline view for objects.

![Original timeline version]({{ site.url }}/assets/timeline_v1.png)

There were a number of problems with this initial implementation that I've been meaning to fix.
First of these was that the TimelineJS script was being loaded from an external URL, rather than
through the asset pipeline. Also, looking at the [Github][timeline-github] account for the Timeline script, showed that
development on that version had stopped, and had switched to the new [TimelineJS3][timeline-js3].

To rectify both of these easily the obvious thing to do was to find a TimelineJS3 gem. I did find one, although
it didn't seem to have been updated for a while. Instead, I used a [fork][gem-fork] that had been modified more recently,
and looked like it had some fixes for the asset pipeline. This worked well in development, but testing in a 
production environment showed that the assets were not loading correctly. To sort that I forked this version and
modified the gem's engine.rb to add the assets to the asset path and to precompile the font files.

{% highlight ruby %}
module TimelineJS3
  module Rails
    class Engine < ::Rails::Engine
      config.assets.paths << config.root.join('vendor', 'assets', 
                                              'javascripts')
      config.assets.paths << config.root.join('vendor', 'assets', 
                                              'stylesheets')
      config.assets.paths << config.root.join('vendor', 'assets', 
                                              'fonts')
      config.assets.precompile << /\.(?:svg|eot|woff|ttf)$/
    end
  end
end
{% endhighlight %}

Using this was then a simple task of adding the gem to the Gemfile:

```
gem 'timelineJS3-rails', git: 'https://github.com/stkenny/timelineJS3-rails.git'
```

Including the JavaScript in the application.js file,

{% highlight javascript %}
//= require timelineJS3
{% endhighlight %}

and the CSS in the stylesheets.

```
@import "timelineJS3";
```

Creating the timeline in the view then only needs:

{% highlight javascript %}
var timeline = new TL.Timeline('time_line', timelineData);
{% endhighlight %}

Here ```timelineData``` is the [JSON format][timeline-json] data listing the objects to be displayed and ```time_line``` 
is the ID of the container div.

There is lots more refactoring of the timeline on the way, but this was a necessary first step.

[timeline-github]:      https://github.com/NUKnightLab/TimelineJS
[timeline-js3]:         http://timeline.knightlab.com/
[gem-fork]:             https://github.com/podemos-info/timelineJS3-rails
[timeline-json]:        https://timeline.knightlab.com/docs/json-format.html




