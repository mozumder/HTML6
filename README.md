_These documents are volatile & updated quite a bit.  Check back often!  And feedback is appreciated._

HTML6 Single-page apps without Javascript
-----------------------------------------

This is a proposal to the HTML standard to allow dynamically loaded single-page apps without Javascript.

This could be done by declaring model objects in the HTML `<HEAD>`:

    <MODEL name="article">
        <FIELD name="headline" type="string">
        <FIELD name="body" type="string">
    </MODEL>

`<A>` elements would specify JSON/XML API endpoints, as well as models that would receive the data:

    <A mref="http://api.mywebsite.com/get-article" receiver="article">Get!</A>

Finally, the DOM is dynamically updated through model references:

    <H1 model="article.headline"></H1>
    <ARTICLE model="article.body"></ARTICLE>

Thus, HTML becomes a templating language, with content residing in model objects that can be dynamically reloaded without Javascript.


Documents
---------

[INTRODUCTION.md](Introduction)
 - A quick why and how.

[Quickstart - Blog.md](Quickstart - Blog)
 - Just want to make a single-page app blog?  Use this document.
 
[More Scenarios.md](More Scenarios)
 - This document contains a bunch of examples to use in the real world.
 
[Model Object.md](Model Object)
 - The spec for the `<MODEL>` element and `MODEL` attributes.

[Data IO.md](Data IO)
 - The spec for data transfer into and out of the models.
 
[Model API.md](Model API)
 - The spec for Javascript access to the model data.
 
[SQL Interface.md](SQL Interface)
 - The spec for accessing model data from HTML.