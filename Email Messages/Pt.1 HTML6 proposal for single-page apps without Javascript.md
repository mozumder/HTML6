Part 1
======

There’s a standard design pattern emerging via all the front-end javascript frameworks where content is loaded dynamically via JSON APIs.  This is the single-page app web design pattern. Everyone’s into it because the responsiveness is so much better than loading a full page - 10-50ms with a clean API load vs. 300-1500ms for a full HTML page load.

Since this is so common now, can we implement this directly in the browsers via HTML so users can dynamically run single-page apps without Javascript? 

My goal would be a high-speed responsive web experience without having to load Javascript.

This could probably be done by linking anchor elements to JSON/XML (or a new definition) api endpoints, having the browser internally load the data into a new data structure, and the browser then replaces DOM elements with whatever data that was loaded as needed. The initial data (as well as standard error responses) could be in header fixtures, which could be replaced later if desired.  

The HTML body thus becomes a templating language, with all the content residing in the fixtures that can be dynamically reloaded without Javascript.

Example web page:
----

    <DOCTYPE html>
    <HTML LANG=“en”>
    <HEAD>
    <FIXTURES lang=“xml”>
        <model class=“MyArticleData”>
            <rsp stat=“ok">
                <article label=“one” id=“1">
                    <headline>"Big News!”</headline>
                    <body>"<p>This is the first article intro.</p><p>This is the second paragraph.</p>"</body>
                </article>
                <article label=“two” id=“2">
                    <headline>"Not so big news"</headline>
                    <body>"<p>This is the <em>second</em> article.</p>"</body>
                </article>
            </rsp>
        </model>
        <model class=“MyImageData”>
            <rsp stat=“ok">
                <image label="Square" width="75" height="75" source="https://mycontentserver.com/image_s.jpg" id=“3"/>
                <image label=“Tall" width=“300" height=“200" source="https://mycontentserver.com/image_l.jpg" id=“4"/>
            </rsp>
            <rsp stat=“loading">
                <image label="Square" width="75" height="75" source="https://mycontentserver.com/loading_image_s.jpg" id=“1"/>
            </rsp>
            <rsp stat=“some_error">
                <image label="Square" width="75" height="75" source="https://mycontentserver.com/error_image_s.jpg" id=“2"/>
                <message
            </rsp>
        </model>
    </FIXTURES>
    </HEAD>
    <BODY>
        <MENU class=“controller”>
            <A href=“http://api.mywebsite.com/api/load-new-article” model=“MyArticleData">Click here to replace the articles with different articles.</A>
            <A href=“http://api.mywebsite.com/api/load-new-image” model=“MyImageData">Click here to replace the picture with a different picture.</A>
        </MENU>
        <MAIN class=“viewer”>
            <ARTICLE class=“center">
                <H1 model=“MyArticleData.rsp.article(label=‘one’).headline” />
                <SPAN model="MyArticleData.rsp.article(label=’one’).body” />
            </ARTICLE>
            <ARTICLE class=“sidebar">
                <H1 model=“MyArticleData.rsp.article(label=’two’).headline” />
                <SPAN model=“MyArticleData.rsp.article(label=’two’).body” />
            </ARTICLE>
            <IMG src=“model:MyImageData.rsp.image(label=‘Square’)#source” width=“model:MyImageData.rsp.image(label=‘Square’)#width” height=“model:MyImageData.rsp.image(label=’Square’)#height”>
        </MAIN>
    </BODY>
    </HTML>

Hope this makes it clear.  

Clicking on a link loads JSON/XML data into a new internal data structure.  This is separate from the DOM.   I have the initial XML fixtures in the `<HEAD>` section here, but these fixtures can be in an external link, or implicitly defined by default within the `<BODY>` elements that contain references to models.  The data structure can be defined implicitly like in this example via XML/JSON fixtures, or perhaps explicitly by SQL statements.

The text sections have `<H1>` and `<SPAN>` tags with a new `MODEL` attribute that defines the content in it based on the loaded data.  This format is declarative, but can approach SQL statement complexity.  You don’t need to have defined controller/view structures, but I did in this example.

There is also a `model:` URI that old attributes and the rest of the page can use to references the loaded data.  Anything in the DOM should be replaceable, including the URL bar and style sheets.  This internal data can be modified by Javascript if needed, separate from modifying the DOM - you could export this data for form processing if you wish.  This internal data can also be push updated by a server or connected directly to a local database for caching or persistence - the browser manages this now, instead of the web developer/Javascript.

Model references are defined so that it can call any object, with the ability to define conditions to access objects, perhaps looping through lists of objects via SQL select statements and using `<TEMPLATE>` tags to create new DOM objects.  There should be standard conventions for common error control.  More advanced link URLs could include SQL statements - `<A href=“http://...“>` becomes `<A href="sql:select from *”>`.  Conventions would be defined for authentication.  The controller’s own `HREF` links could also be changed.  A link could update multiple model structures.  

Anyways, what do you think about this?  I think something like this could eliminate a lot of Javascript.  These javascript frameworks are all trying to do this, but none of them do it easily and they’re always being redesigned.  Something should be done to standardize on it at a higher level.  There’s a tremendous speed advantage in this, and we shouldn’t have to program in Javascript for such a common design pattern.  Web pages should be loadable with 60fps speeds and should be as responsive as native apps.


