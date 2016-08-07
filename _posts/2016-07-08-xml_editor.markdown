---
layout: post
title:  "XML Editing Part 1"
date:   2016-08-07 17:53:00
categories: rails javascript
---

One of the main features of DRI is our support for multiple metadata standards. Although user's can ingest
DC, MARC, MODS and EAD metadata in XML format it has only been possible to edit Dublin Core records. To edit
MARC or MODS required modifying the XML locally and replacing the object metadata with the edited version.
A cataloguing tool that would allow user's to make these changes in the browser is something that was often
requested. Building web-forms to cater for all these standards has always seemed like too difficult a task.
One possible solution is to enable them to simply edit the XML metadata directly, as they would be doing
locally. This would require adding an XML editor to the Repository UI. The [JQuery XML editor][ucla-xmleditor] 
created as part of the Carolina Digital Repository, seemed like it would be exactly what I needed.

To try it out the first step was to gemify it for easier inclusion into the Repository. 

First I created a skeleton gem. There are different ways of doing this, but I used bundler.

```
bundle gem jquery-xmleditor-rails
```

This creates the basic directory structure for the gem. To this I needed to add the source code for the editor,
 which I downloaded from Github. The javascript goes in the app/assets/javascripts directory.


```
cd jquery-xmleditor-rails
```

```
mkdir -p app/assets/javascripts/jquery-xmleditor
```

```
cp ../jquery.xmleditor.js app/assets/javascripts/jquery-xmleditor/jquery.xmleditor.js.erb
```

The editor has a number of dependencies, which in Github are stored in the lib directory. I put these
in a vendor directory in the assets javascripts folder, like so:

```
ls app/assets/javascripts/jquery-xmleditor/vendor/
```

```
ace  cycle.js  jquery.autosize-min.js  json2.js
```

I then created an index.js to load the javascript file and its dependencies when its included into the app.

```
cat app/assets/javascripts/jquery-xmleditor/index.js
```

{% highlight javascript %}
//= require jquery-xmleditor/jquery.xmleditor
//= require_tree .
{% endhighlight %}

For the stylesheet its the simple task of copying the css file into the app/assets/stylesheets directory:

```
cp ../jquery.xmleditor/css/jquery.xmleditor.css app/assets/stylesheets/jquery.xmleditor.scss
```

One problem I ran into was that the javascript tries to load one of the dependencies, cycle.js, from a local
URL. So I needed the gem to make this available. To be honest I'm not sure this is the correct, or best approach,
but to do this I copied the cycle.js file to a public/lib directory. We now need to make the gem serve this file. 
This requires modifying some files in the gem's lib directory.

First, lib/jquery-xmleditor-rails.rb

{% highlight ruby %}
module JQuery
  module XmlEditor
    module Rails
      require 'jquery/xmleditor/rails/engine' if defined?(Rails)
    end
  end
end
{% endhighlight %}

Then lib/jquery-xmleditor/rails/engine.rb

{% highlight ruby %}
module JQuery
  module XmlEditor
    module Rails
      class Engine < ::Rails::Engine

        # Initializer to combine this engines static assets 
        # with the static assets of the hosting site.
        initializer "static assets" do |app|
          app.middleware.insert_before(
          	::ActionDispatch::Static, 
          	::ActionDispatch::Static, "#{root}/public")
        end

      end
    end
  end
end
{% endhighlight %}

Other than modifying a few more files, such as the version.rb and the gemspec, this was all that was required to create
a functioning gem for the editor. The full code for the gem can be found [here][gem-github].

[ucla-xmleditor]:  https://github.com/UNC-Libraries/jquery.xmleditor
[gem-github]:      https://github.com/stkenny/jquery-xmleditor-rails





