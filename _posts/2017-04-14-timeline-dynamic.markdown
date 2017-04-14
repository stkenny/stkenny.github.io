---
layout: post
title:  "Dynamic Timeline"
date:   2017-04-14 17:15:00
categories: rails javascript
---

I should start with a disclaimer that this isn't really the use intended for a timeline. From 
the [TimelineJS][timeline-js3] website you can see that it is really meant to tell a story, and should probably
have only around 20 or so slides. We are using a timeline as essentially another way to browse
objects, or to visualise search results. Because this use case isn't supported out of the box
I needed to find a way of dynamically loading objects as a user scrolled back and forth, as clearly
loading all the objects at once would not be possible. Ideally something like this could be
handled with Ajax requests, retrieving data when needed. But our browse/search already supports
paging of results, so a simpler, if slightly hacky way, was to make use of this.

First some variables need to be accessible to the javascript for creating the timeline. This can be done 
with data attributes.

{% highlight ruby %}
content_tag "div", id: "dri_timeline_id", style: "height:550px", 
data: { total_count: @response.total_count, 
        total_pages: @response.total_pages, 
        current_page: @response.current_page, 
        timeline_data: @timeline_data }
{% endhighlight %}

The attributes provide the total number of pages of results and the current result page that is being viewed. 
Normally when the end of the timeline is reached the next slide navigation is disabled. What I wanted is to 
re-enable this but have it retrieve the next page of results rather than the next slide. That is
what the javascript below does.

If we are viewing the last slide the next slide navigation is enabled, but the title and description are set
to show that it will now load more results. If the navigation is clicked URL query parameters are set (tl_page) 
to the next page and loading of the new URL is triggered.

{% highlight javascript %}
timeline.on("nav_next", function(data) {
    var slides = data.target._slides;

    for (var i = 0; i < slides.length; i++) {
      if (slides[i].active == true) {
        index = i;
      }
    }

    if (index == slides.length - 1 && currentPage < totalPages) {
      if ($( ".tl-slidenav-next .tl-slidenav-title")
        .html() == "More results") {
          query = setUrlEncodedKey("tl_page", currentPage + 1);
          query = setUrlEncodedKey("direction", "forward", query);
        
          $(location).attr('href', query);
      } else {
        $( ".tl-slidenav-next" ).show();
        $( ".tl-slidenav-next .tl-slidenav-title")
           .html("More results");
        $( ".tl-slidenav-next .tl-slidenav-description")
           .html("Load more timeline objects.");
      }
    }
  });
{% endhighlight %}

![More results navigation]({{ site.url }}/assets/timeline_next.png)

Allowing previous pages to be loaded is almost exactly the same, except that we are moving in the 
opposite direction.

{% highlight javascript %}
timeline.on("nav_previous", function(data) {
    var slides = data.target._slides;

    for (var i = 0; i < slides.length; i++) {
      if (slides[i].active == true) {
        index = i;
      }
    }
    if (currentPage > 1 && index == 0) {
      if ($( ".tl-slidenav-previous .tl-slidenav-title")
        .html() == "Previous results") {
          query = setUrlEncodedKey("tl_page", currentPage - 1);
          query = setUrlEncodedKey("direction", "back", query);

          $(location).attr('href', query);
      } else {
        $( ".tl-slidenav-previous" ).show();
        $( ".tl-slidenav-previous .tl-slidenav-title")
          .html("Previous results");
        $( ".tl-slidenav-previous .tl-slidenav-description")
          .html("Load previous timeline objects.");
      }
    }
  });
{% endhighlight %}

![Previous results navigation]({{ site.url }}/assets/timeline_previous.png)

One issue with loading the previous page of results is that the timeline will start from the first slide
rather the last, which is not really what you might expect when navigating the timeline. I haven't found a
totally satisfactory solution for that one. The best I have so far is to tell the timeline to start from the end.
Now when the previous page loads it automatically scrolls to the last slide.

{% highlight javascript %}
if (direction == "back") {
  timeline.goToEnd();
  $( ".tl-slidenav-next" ).trigger("click");
}
{% endhighlight %}

[timeline-js3]:         http://timeline.knightlab.com/
