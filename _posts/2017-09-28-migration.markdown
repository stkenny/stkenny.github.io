---
layout: post
title:  "Another Migration"
date:   2017-09-28 12:17:00
categories: rails repository
---

For the last two months I've been looking at another major migration of the DRI's data models.
The last migration happened just before the official launch (yes, that was not a great idea)
when we moved from [Fedora][fedora] 3 to Fedora 4. As well as modifications to the data model code,
that migration also required re-ingesting all the objects in the repository to a new Fedora 4 server.
With a relatively small number of collections this was a possible, if lengthy operation.

The reason for this latest migration is a combination of factors. First there has been a change in
the [underlying storage][fedora-release] used by the latest versions of Fedora. Fedora 4 uses Modeshape as a data
store and initially this in turn used Infinispan. Recently Modeshape has dropped support for Infinispan
meaning that the latest versions of Fedora (4.7+) now use Modeshape with RDBMS backends. Although
this is a welcome change, it does pose a problem, in that the migration path is a full backup and
restore of Fedora. Basically take everything out and then re-ingest it again. To date I have not managed
to successfully complete a full backup and restore in this way.

A second is really a result of legacy problems to do with Fedora 3. A lot of our code involved
avoiding making use of Fedora where possible for performance reasons. So, for example, all queries use Solr
and we do not store the binary files (assets) in Fedora datastreams, but rather direct to disk. Another
development was our use of [Moab][moab] for preservation storage of metadata and binaries. Doing this has meant
that Fedora is purely being used as on object store, but we are not making any use of its other features.
It made sense then to take this migration path problem as an opportunity to try out another approach.

ActiveFedora, the library used to interact with Fedora is described as 'loosely based on "ActiveRecord"'.
The most obvious and potentially straightforward migration then would be to move instead to just
using ActiveRecord directly. So that is what has been done. Currently a simplified version of our stack
looks like this:

![Current stack]({{ site.url }}/assets/current_stack.png){:height="50%" width="50%"}

Following the migration it will be as below. Although this might look like not much has changed, this is a significant
step. Hopefully this will lead to better performance as well as a more understandable and more easily managed infrastructure.
It should also allow us to keep in line with future Rails and gem dependency releases.

![Future stack]({{ site.url }}/assets/future_stack.png){:height="50%" width="50%"}

More details on how the migration is being done to follow.

[fedora]:         http://fedorarepository.org/
[fedora-release]: https://wiki.duraspace.org/display/FF/Fedora+4.7.0+Release+Notes
[moab]:           http://journal.code4lib.org/articles/8482
