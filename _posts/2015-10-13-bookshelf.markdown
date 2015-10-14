---
layout: post
title:  "DRI DIY: Building a bookshelf"
date:   2015-10-13 15:19:33
categories: rails javascript css
---
During the run up to the DRI launch, to take a break from ingesting collections, I was playing around with a JavaScript page-turner. I re-visited that recently to make it work through the repository API, rather than connecting directly to the Solr service. This was so that it could be used as a standalone application outside of the local network. I took this as an opportunity to play around with some JavaScript and CSS in Rails. I guess it turned into more of an exercise in self-assembly rather than DIY, as I pieced a lot of existing code together.

[Turn.js][turnjs] is a JavaScript library that makes it easy to create a nice looking flip book. With this library the code needed to create and display a book with hardcoded pages was straightforward. The idea was to build the book dynamically using images from a collection stored in the DRI repository. To do this it was necessary to have a way to translate page numbers to object IDs. For at least one of our collections this was possible thanks to a field in the object metadata that could be mapped to the page number:

{% highlight xml %}
<dc:identifier>1915_0002</dc:identifier>
{% endhighlight %}

By using a Ruby template string, the identifier can be obtained from the page number quite easily:

{% highlight ruby %}
page = 2
template = "1915_%04d"

identifier = template % page
{% endhighlight %}

The object's ID can be retrieved by submitting a query to the repository containing this identifier. Once we have the ID the list of available images for that object can be retrieved with another API call. The repository returns a JSON list of file URLs. Using the URL from this list that corresponds to the format needed the page can be created and added to the book.

![The page-turner]({{ site.url }}/assets/flipbook.png)

Another post, <https://github.com/blasten/turn.js/wiki/Making-pages-dynamically-with-Ajax>, gave the steps needed to dynamically add pages to the book. An AJAX call is made to the page turner app controller that handles all the steps above, and returns a DIV containing the page URL.

{% highlight ruby %}
def page
    book_id = params[:book_id]
    page = params[:id]

    # Find the PageTurner model for this book
    turner = PageTurner.find(book_id)

    # Use the model to retrieve the image 
    # URL to use for the given page number
    url = turner.page_url(page) if turner

    # Return the HTML to add to the flip book
    render text: "<div><img src='#{url}'/></div>"
end
{% endhighlight %}

For no reason other than I thought it might look nice I started looking at how to add a bookshelf to hold the 'books'. Finding and following this blog post, <http://www.hmp.is.it/making-a-fancy-book-using-html5-canvases/>, showed how it could be done. It makes use of CSS, HTML5 and JavaScript. Taking the code and integrating it with the Rails asset pipeline allowed the book to be displayed on a 'shelf'. Clicking the book links through to the flip book.

![The bookshelf]({{ site.url }}/assets/shelf.png)

The final stage was to try out [Heroku][heroku] to deploy the app as a demo. You should be able to access it at <https://dry-savannah-3204.herokuapp.com/books>. I'm using the free service, so it might not always be available.

Obviously this isn't the cleanest solution, and it could do with a lot of tweaking, but as an exercise in the Rails pipeline it was quite useful.

[turnjs]:            https//turnjs.com
[heroku]:            https://dashboard.heroku.com/
