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

[Introduction](https://github.com/mozumder/HTML6/blob/master/INTRODUCTION.md)
 - A quick why and how with rough examples and ideas.

[Quickstart - Blog](https://github.com/mozumder/HTML6/blob/master/Quickstart%20-%20Blog.md) _currently empty_
 - Just want to make a single-page-app blog?  Use this document.
 
[More Scenarios](https://github.com/mozumder/HTML6/blob/master/More%20Scenarios.md) _currently empty_
 - This document contains a bunch of examples to use in the real world.
 
[Model Object](https://github.com/mozumder/HTML6/blob/master/Model%20Object.md) _currently empty_
 - The spec for the `<MODEL>` element and `MODEL` attributes.

[Data IO](https://github.com/mozumder/HTML6/blob/master/Data%20IO.md) _currently empty_
 - The spec for data transfer into and out of the models.
 
[Model API](https://github.com/mozumder/HTML6/blob/master/Model%20API.md)
 - The spec for Javascript access to the model data.
 
[SQL Interface](https://github.com/mozumder/HTML6/blob/master/SQL%20Interface.md) _currently empty_
 - The spec for accessing model data from HTML using SQL.