---
layout: post
title:  "App to Engine"
date:   2018-01-01 17:30:00
categories: rails repository
---

Some time ago I put together an application for batch ingesting data into the DRI repository.
This app was to replace the existing command-line tool. Although I did use the app a few times,
it never really became part of the production infrastructure. At least, not in a way that would make
it available to end users. I decided that the easiest way forward was to integrate the functionality
into the repository, but without having to add all the code directly into the codebase. To do this I 
converted the app into a Rails engine.

First step, create the engine skeleton:

```
rails plugin new dri_batch_ingest -T --mountable --full
```

This creates the basic directory structure and files needed by the engine. Once this exists the next task
is to copy the source files from the app into the correct locations in the engine. Because this is to be a
namespaced engine the main directories are:

```
app/controllers/dri_batch_ingest
app/models/dri_batch_ingest
lib/dri_batch_ingest
```

The original application had a number of dependencies for the frontend. The correct place to put these in an engine
is the gemspec file, ```dri_batch_ingest.gemspec```.

```
s.add_dependency "rails", "~> 4.2.10"
s.add_dependency 'kaminari'
s.add_dependency 'fuelux-rails-sass'
s.add_dependency 'underscore-rails'
s.add_dependency 'iconv'
s.add_dependency 'filesize'
```

Two extra dependencies that could not be added to the gemspec are ```avalon_ingest``` and ```browse-everything```.
The first of these bundles together some functionality for performing the batch ingest, the other allows for browsing
various cloud-storage endpoints. I'm pulling these dependencies from github, which is not allowed within the gemspec, so 
these had to be added to the Gemfile.

```
gem 'avalon_ingest', git: 'https://github.com/stkenny/avalon_ingest'
gem 'browse-everything', git: 'https://github.com/stkenny/browse-everything.git', branch: 'feature/per_user'
```

To load these dependencies we need to edit the engine file, ```lib/dri_batch_ingest/engine.rb```:

{% highlight ruby %}
# dependencies
require 'underscore-rails'
require 'fuelux-rails-sass'
require 'browse_everything'

module DriBatchIngest
  class Engine < ::Rails::Engine
    isolate_namespace DriBatchIngest

    # use rspec for testing
    config.generators do |g|
      g.test_framework :rspec
    end

    # this allows the migrations to stay in the engine
    # rather than having to copy them into the app the 
    # engine will be used in
    initializer :append_migrations do |app|
      unless app.root.to_s.match root.to_s
        config.paths["db/migrate"].expanded.each do |expanded_path|
          app.config.paths["db/migrate"] << expanded_path
        end
      end
    end

  end
end
{% endhighlight %}

and then require the javascript in the asset pipeline, ```app/assets/javascripts/dri_batch_ingest/index.js```

```
//= require underscore
//= require fuelux
//= require_tree .
//= require browse_everything
```

The namespacing of the engine means that migrations also look a bit different from a normal application. They have a prefix, in this case ```dri_batch_ingest_```. Here is an example:

{% highlight ruby %}
class CreateTableIngestBatch < ActiveRecord::Migration

  def change
    create_table :idri_batch_ingest_ingest_batches do |t|
      t.string :email
      t.string :collection_id
      t.integer :user_ingest_id
      t.text :media_object_ids
      t.boolean :finished, :default => false
      t.boolean :email_sent, :default => false
      t.timestamps
    end
  end
end
{% endhighlight %}

This engine now encapsulates all the batch ingest functionality needed by the main DRI repository application. You can see the full engine in the DRI [github](https://github.com/Digital-Repository-of-Ireland/dri-batch-ingest).
