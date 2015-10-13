---
layout: post
title:  "DRI DIY: Building a bookshelf"
date:   2015-10-13 15:19:33
categories: rails javascript css
---
During the run up to the DRI launch, in order to give myself a break from ingesting collections, I was playing around with a JavaScript page-turner. I revisited that recently to try and make it work through the repository API, rather than connecting directly to the Solr service. I took this as an opportunity to play around with some JavaScript and CSS in Rails. I guess it turned into more of an exercise in self-assembly rather than DIY, as I pieced a lot of existing code together.

The main library used is [Turn.js][turnjs], a JavaScript library that makes it easy to create a nice looking flip book. Although the code for this was straightfoward, the problem is with our current data models. A work in progress is to have a way to define relationships between objects such as 'is a page of', 'next page', or 'previous page' and so on. I needed a way of finding pages and determining order. Happily, for at least one of our collections, the page number could be determined from the identifier used in the metadata:

{% highlight xml %}
<dc:identifier>1915_0002</dc:identifier>
{% endhighlight %}

We can translate from a page number to the identifier with a simple Ruby template String:

{% highlight ruby %}
page = 2
template = "1915_\04d"

identifier = template % page
{% endhighlight %}

Getting the page image is then a two step process. First, query solr for the ID of the object with the correct identifier and then retrieve the list of available surrogates for that object. We can then choose the surrogate file that we want to use from the returned list, in this case 'lightbox_format'. The Turn.js library pre-fetches the next 5 pages if they are not already in the model, so load time of the images is at least partially minimised.

![The page-turner]({{ site.url }}/assets/flipbook.png)

The bookshelf was then added. This was done by following this blog post, [http://www.hmp.is.it/making-a-fancy-book-using-html5-canvases/]. It makes use of CSS and JavaScript to create the bookshelf. Clicking on a book should bring you to the flipbook page.

![The bookshelf]({{ site.url }}/assets/shelf.png)

The final stage was to try out [Heroku][heroku], to deploy the app as a demo. You should be able to access it at [https://dry-savannah-3204.herokuapp.com/books].

Obviously this isn't the cleanest solution, and it could do with a lot of tweaking, but as an exercise in the Rails pipeline it was quite useful.

[turnjs]:            https//turnjs.com
[heroku]:            https://dashboard.heroku.com/
