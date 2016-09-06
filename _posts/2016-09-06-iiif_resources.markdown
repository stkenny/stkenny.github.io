---
layout: post
title:  "IIIF Resources"
date:   2016-09-06 16:27:00
categories: rails iiif
---

I'm currently working on adding [IIIF][iiif] support to the DRI web application. The steps that must be
completed for this, taken from the IIIF website are:

1. Deploy an image server that supports the IIIF Image API
2. Publish metadata about your image-based objects that complies to IIIF Presentation API
3. Deploy and integrate software that allows you to discover and display IIIF-compliant image resources 

## Image server
The first step is to add support for the IIIF image API. As we already serve images from the web application 
it is a case of adapting the existing server, rather than deploying a new separate image server.
To do this I will be using the [RIIIF][riiif] gem from the Curation Experts github repository. This gem provides
a IIIF image server as a Rails engine. The README describes how to integate this with a Hydra application, although this
will need modification to work with our data models.

## Image-based object metadata
The images need to provide IIIF presentation API compatible metadata. This is in the form of a JSON manifest.
All of our objects have metadata associated with them in the XML format that was ingested, so it is a case
of translating this into the expected format, and making it accessible from a route in the web application.
There is a gem in the [IIIF github][iiif-github] that I am using to perform this translation called 
[osullivan][osullivan].

Unfortunately the documentation for this is pretty minimal, so actually creating a manifest is a bit hit and miss.
Useful resources to help are the IIIF documentation on the [presentation API][iiif-presentation], in particular
the example manifest given in the appendices.

Also very helpful is the [Tripoli][tripoli] validator for IIIF presentation API documents. The error messages
very clearly point out what needs to be changed in the manifest. I had some difficulty getting it to install, 
but luckily there is an online version that works: https://validate.musiclibs.net/

## Discover and display
Lastly the fun bit, the viewer. I'm planning on using [Mirador][mirador] as it seems to be a popular choice.
The decision to be made is how to go about the integration, either deploy on a separate server and link out
to it to launch the viewer, or more closely integrate it into the application itself. For integrating Mirador
into the application I've been trying to create a Mirador gem. Although this seems to work it does require a
lot of javascript and other resources, so it might be cleaner and more performant to keep the viewer on a separate
host.

[iiif]:              http://iiif.io/
[riiif]:             https://github.com/curationexperts/riiif
[iiif-github]:       https://github.com/IIIF
[osullivan]:         https://github.com/IIIF/osullivan
[iiif-presentation]: http://iiif.io/api/presentation/2.1/#terminology
[tripoli]:           http://tripoli.readthedocs.io/en/latest/
[mirador]:           http://projectmirador.org/
