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

Clicking on a link loads JSON/XML data into a new internal data structure.  This is separate from the DOM.   I have the initial XML fixtures in the <HEAD> section here, but these fixtures can be in an external link, or implicitly defined by default within the <BODY> elements that contain references to models.  The data structure can be defined implicitly like in this example via XML/JSON fixtures, or perhaps explicitly by SQL statements.

The text sections have <H1> and <SPAN> tags with a new MODEL attribute that defines the content in it based on the loaded data.  This format is declarative, but can approach SQL statement complexity.  You don’t need to have defined controller/view structures, but I did in this example.

There is also a “model:” URI that old attributes and the rest of the page can use to references the loaded data.  Anything in the DOM should be replaceable, including the URL bar and style sheets.  This internal data can be modified by Javascript if needed, separate from modifying the DOM - you could export this data for form processing if you wish.  This internal data can also be push updated by a server or connected directly to a local database for caching or persistence - the browser manages this now, instead of the web developer/Javascript.

Model references are defined so that it can call any object, with the ability to define conditions to access objects, perhaps looping through lists of objects via SQL select statements and using <TEMPLATE> tags to create new DOM objects.  There should be standard conventions for common error control.  More advanced link URLs could include SQL statements - <A href=“http://...“> becomes <A href="sql:select from *”>.  Conventions would be defined for authentication.  The controller’s own “href” links could also be changed.  A link could update multiple model structures.  

Anyways, what do you think about this?  I think something like this could eliminate a lot of Javascript.  These javascript frameworks are all trying to do this, but none of them do it easily and they’re always being redesigned.  Something should be done to standardize on it at a higher level.  There’s a tremendous speed advantage in this, and we shouldn’t have to program in Javascript for such a common design pattern.  Web pages should be loadable with 60fps speeds and should be as responsive as native apps.


Part 2
======

Lots of people seem to be picking up on the idea of a <MODEL> element.  In my original example, I mentioned that models could be defined implicitly via the fixtures (or HTML BODY), or something else like SQL statements.  We could also define them directly in the <HEAD> via explicit HTML <MODEL> elements.

So, in my initial example, I defined fixtures with:

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
            </rsp>
        </model>
    </FIXTURES>

This implies a set of models and so the browser creates those models.

A version with explicit model definitions via <MODEL> elements would be:

    <HEAD>
    <MODEL class=“MyArticleData”>
        <MODEL class=“rsp” type=“response">
            <MODEL class=“article”>
                <PROPERTY name=“label” type=“string”>
                <MODEL class=“headline” type=“string”>  
                <MODEL class=“body” type=“string">
            </MODEL>
        </MODEL>
    </MODEL>
    <MODEL  class=“MyImageData”>
        <MODEL class=“rsp" type=“response">
            <MODEL class=“image”>
                <PROPERTY name=“label” type=“string”>
                <PROPERTY name=“width” type=“integer”>
                <PROPERTY name=“height” type=“integer”>
                <PROPERTY name=“source” type=“url”>
            </MODEL>
        </MODEL>
    </MODEL>
    <FIXTURES>
        … fixtures go here (XML or JSON format)
    </FIXTURES>
    </HEAD>

The internal model data structures are constructed based on these model definitions, which, again, may be implicitly or explicitly defined.  The internal models are canonical and strongly typed, and should be as equivalent to SQL as possible.  Also, these are HTML elements - whereas my previous example model fixtures are actually XML.

A <MODEL> is equivalent to an SQL TABLE.  Both the model’s TYPE attribute and <PROPERTY> elements would be the equivalent of an SQL COLUMN.  For example, the “article” model has a TYPE=“string”, and this means the article model has a default field that is equivalent of an SQL VARCHAR field.  We can add more fields via <PROPERTY> elements, as done in the example.  The “image” model in this example doesn’t have a default field, but we added several via <PROPERTY> elements.  All models should have at least one default and unique primary key - ID.  I don’t show it here, but this is where we would have foreign key references as well, via a <REFERENCE> elements.  This example uses child model definitions to make it clear.

Certain model types are used to define behavior. The “response” type for the “rsp” model indicates to the browser that this is going to be an API’s response, so let’s prepare for that.  Strings could have maxLengths, etc.

The fun stuff starts to happen with these model definitions. This is where you would have more advanced apps that go beyond simple CRUD.  I mentioned in the original proposal that you could process these objects in Javascript if you wish, and will expand on that here.  

We could assign Javascript event callbacks to these models or properties that perform actions.  There would be a whole set of events, similar to what the HTML presentation layer has already (onMouseOver, onKeyPress, etc.). For model objects, it would be things like onLoad, onExpire, etc.. far too numerous to list here in this early concept stage.  It would include event message objects and bubbling, like the rest of Javascript’s event system.

For example, if I wanted to translate my JSON Markdown text object into HTML for presentation, I could do that in a callback function attached in the HTML:

	<MODEL class="MyArticleData” onLoad=“MarkdownToHTML”>

They could be attached in Javascript:

	models.MyArticleData.onLoad=function(){MarkdownToHTML};

or:

	models.MyArticleData.addEventListener(“onLoad”, MarkdownToHTML)

You could access model objects via a Javascript API:

	models.MyArticleData.rsp.get("stat=‘ok’”).article.get(“id=1").headline = “Dewey Defeats Truman”

or directly via arrays on the primary key:

	models.MyArticleData.article[1].headline = “Nope”

(I collapsed in the “rsp” model here into MyArticleData as a feature of the “response” model type.)

You could apply the above Markdown converter to just the article "body” model object:

	models.MyArticleData.article.body = function(){MarkdownToHTML};

You could do things like subscribe to a push API server:

	models.MyArticleData.pushSubscribe(“http://api.mynewsserver.com/user-id-5678”)

or:

	<MODEL class="MyArticleData” src=“http://api.mynewsserver.com/user-id-5678”>

And whenever you get a new message, you can update the page:

	models.MyArticleData.onPush=function(){UpdateMessageWindow}

Form fields interact with models directly:

	<INPUT type=“text” model=“MyArticleData.article.headline”>

You don’t need a <FORM> element for this interaction with the model, as it updates live. You do need it if you want to send the data to the server via a <BUTTON>, and we could add a <FIELD> element for sending model data:

	<FORM action=“http://api.mynewsserver.com/edit-article”>
		<FIELD model=“MyAppData.api_key”>
		<FIELD model=“MyArticleData.article.id”>
		<FIELD model=“MyArticleData.article.headline”>
		<FIELD model=“MyArticleData.article.body”>
		<BUTTON type=“submit” >Edit Article</BUTTON>
	</FORM>

If you want to receive data into a model as well:

	<FORM action=“http://api.mynewsserver.com/load-article” model=“MyArticleData">

With this, you could create basic but highly-responsive single-page CRUD apps without Javascript.

For more custom stuff in Javascript, there would be an equivalent to XMLHttpRequest API, except for model interactions:

	var request = new ModelHttpRequest();
	request.open(‘GET’, 'http://api.mynewsserver.com/get-latest-article', true);
	request.receiverModel(MyArticleData)
	request.send()

If you wanted to send a model via Javascript instead of an HTML <FORM>:

	var request = new ModelHttpRequest();
	request.open(‘POST’, 'http://api.mynewsserver.com/edit-article', true);
	request.sendModelData([MyAppData.api_key, MyArticleData])
	request.send()

There would be lots of features here, such as translating API names to model names.

If you always wanted to send new objects:

	<MODEL class="MyArticleData” onChange=“mySendFunction”>

This is also where you could define custom non-default response handlers if you wish:

	<MODEL class="MyArticleData” onResponse=“myResponseHandler”>

As for presentation, use the <TEMPLATE> element to show model objects in lists:

	<UL class=“headLines”>
		<TEMPLATE model=“MyArticleData.article">
			<LI model=“this.headline”></LI>
		</TEMPLATE>
	</UL>

This would be the equivalent to "block" structures in other templating languages, with “this” accessors available within a template block.  

Maybe you want infinite scroll:

	<UL class=“headLines”>
		<TEMPLATE model=“MyArticleData.article">
			<LI model=“this.headline” onShow=“loadMoreArticles”></LI>
		</TEMPLATE>
		<TEMPLATE model=“MyArticleData.article">
			<LI model=“this.headline”></LI>
		</TEMPLATE>
		<TEMPLATE model=“MyArticleData.article">
			<LI model=“this.headline”></LI>
		</TEMPLATE>
	</UL>

Each reference to the same model cycles through model objects here.

Advanced users might use the object store for caching purposes:

	<MODEL class="MyArticleData” src=“http://api.mynewsserver.com/user-id-5678” expire=300>

This would invalidate records after 300 seconds.  The SRC connection could also push a cache invalidation. 

Caching is a bit complicated and I want to keep this high-concept design note quick & simple, so will leave it for now.

Anyways, I believe that in all cases, this entire approach would be a lot simpler than creating custom HTML components the way the other Javascript frameworks force you to do.  I just want my articles updated. I should’t have to make complicated web components just to do that.  This a huge problem with the current frameworks, and what these custom components are doing is actually breaking HTML’s semantics.  An <NG-ARTICLE> breaks semantics, but <ARTICLE> doesn’t, so can we just stick with <ARTICLE>?

Thanks for listening everyone.. and if you vote for this, all your dreams will come true.

Part 3
======

People are freaking out about the proposal idea of using SQL statements directly to populate the browser’s internal model data store, when I mentioned using <A href="sql:select from *”>

a few things about this:

1. The security issues aren’t as big when operating on local data for a local app, like a single-player game on a mobile device.

2. There are millions of non-interactive websites with static content that also don't need to worry about complex security.  You still have the same level of table read permissions, just disallow writes by default.

3. This was designed with a front-end sandbox perspective, not a back-end server perspective.  For more complex authorization requirements, I expect a use model where the server transforms SQL statements. 

For example:

	SELECT first_name, last_name FROM users;

would be transformed into:
	
	SELECT first_name, last_name FROM users WHERE manager=“Boss Man”;

The back-end web/app server does this type of transform before hitting the database.  The server basically offers the client a subset of the database schema.  The client might not even see nor be aware of the “manager” column, for example.  The front-end user just sees this limited sub-schema, and happily operates only that data.  Since it’s up the the server to do this type of transform, the server might not even be using an SQL RDBMS, and it might transform/interface into something else entirely.  

4. This SQL interface would be an optional spec.  On the internet, you would most likely be using JSON APIs at first anyways.  But I would recommend something like this because this was designed to eliminate a layer of back-end when using a local database app, as well as eliminating the need to use Javascript for such local database accesses.  

Anyways thanks for listening.  If you vote for this, you can use me as a reference for 4 years of HTML6 experience.

