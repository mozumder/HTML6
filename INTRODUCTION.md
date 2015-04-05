
Problem
-------

There’s a standard design pattern emerging in large front-end Javascript frameworks (Angular, Ember, React, etc.) where content is loaded dynamically through JSON APIs and displayed on the page.  This is commonly known as the single-page web app.  

Web developers do this to help the user's web browsing experience.  Normally, whenever a user clicks on a link, the browser loads a completely new HTML document.  Meanwhile, on a typical website, the pages share a common HTML document structure.  As a user traverses a site, the server has to generate the same parts of the page over and over, and the browser has to reload that same data over and over.  This wastes time and power, and can result in a noticeable lag when interacting with a website.  So, web developers use these Javascript frameworks to cut down on that lag.  When a user clicks a link, these Javascript frameworks only load certain parts of the HTML document.  Since the document structure typically doesn't change for a site, they can just reload the elements that do change. Doing this shaves several milliseconds when interacting with a webpage, which can be a critical to the user's experience.

Meanwhile, there are several obvious problems with this approach.  Every web page now has to include a huge Javascript framework that the browser has to load and compile.  These frameworks can be several hundred KB for a page.  This takes time and power.  These Javascript frameworks aren't easy to use, and have different features and capabilities.   They’re all trying to figure it out and in a state of flux. Angular is being redesigned and will break backwards compatibility for 2.0.  Ember is also being heavily upgraded.  Backbone & React doesn’t do the full flow, as you still need to bring in other frameworks for things like persistent data modeling and binding.  There are probably new frameworks popping up weekly as well.

Additionally, a big problem is Javascript itself.  Should millions of web developers be forced to learn Javascript, a framework, and an associated templating language, if they want a speedy and responsive website out-of-the-box?  This is a huge barrier for beginners, so right now they’re stuck with slower full page reloads, because by default, HTML loads full web pages.

So there's a real, heavy cost associated with these frameworks.  This cost needs to be reduced for a successful web.


Proposal
--------

Since this web design pattern is so common now, browsers should implement this directly via enhancements to HTML.  Users can then dynamically run single-page apps without Javascript.

This could be done by declaring model objects in the HTML `<HEAD>`:

    <MODEL name="article">
        <FIELD name="headline" type="string">
        <FIELD name="body" type="string">
    </MODEL>

`<A>` elements would specify JSON/XML API endpoints and models that receive data:

    <A mref="http://api.mywebsite.com/get-article" receiver="article">Get!</A>

Finally, the DOM is dynamically updated through model references:

    <H1 model="article.headline"></H1>
    <ARTICLE model="article.body"></ARTICLE>

Initial data, as well as standard error responses, could be in `<BODY>` elements as well as `<HEAD>` fixtures.  

Thus, HTML becomes a templating language, with content residing in model objects that can be dynamically reloaded without Javascript.

Example
-------

    <!DOCTYPE html>
    <HTML lang="en">
    <HEAD>
        <MODEL name="article">
            <FIELD name="headline">
            <FIELD name="body">
            <MODEL name="image">
                <FIELD name="width" type="integer">
                <FIELD name="height" type="integer">
                <FIELD name="src" type="url">
                <FIXTURE type="json">
                    {
                     "width": "200",
                     "height": "200",
                     "src": "https://mycontentserver.com/first_image.jpg",
                    }
                </FIXTURE>
            </MODEL>
        </MODEL>
    </HEAD>
    <BODY>
        <MENU>
            <A href="article-2.html" mref="http://api.mywebsite.com/article-2" receiver="article">Click here to replace the article with a different article.</A>
        </MENU>
        <MAIN>
            <ARTICLE>
                <H1 model="article.headline">Big News!</H1>
                <SPAN model="article.body"><P>This is the initial article.</P></SPAN>
                <IMG model="article.image">
            </ARTICLE>
        </MAIN>
    </BODY>
    </HTML>

In this example, the `<MODEL>` element defines a data table, and `<FIELD>` elements define fields within that table.  A model table may contain multiple rows, optionally accessible by an index.

Initial model data can be loaded in from one of 3 places: `<FIXTURE>` elements in a `<MODEL>`, external fixture URLs, or from elements in the page that contain model references.  In this example, the `article` model loads initial `headline` and `body` data from the `<H1>` and `<SPAN>` elements, while the `image` model loads initial data from the `<FIXTURE>` element.  The overall schema can be defined explicitly by `<MODEL>` elements, implicitly by `<FIXTURE>` elements, or even through SQL statements.

Clicking on a link loads JSON data from an API endpoint specified in the `MREF` property into a destination model specified in the `RECEIVER` property.  Model objects are separate from the DOM. The `HREF` property remains as a canonical URL for the page, and can be used by older browsers that don't support dynamic page updates, or shared to other users to recall the site.  The `HREF` would be placed in the URL bar via `pushState`, and can change during navigation.  The `HREF` is also optional, and a site can operate fully within the same URL if the site's author wishes.

The text sections in this example have `<H1>` and `<SPAN>` elements with a new `MODEL` attribute.  This `MODEL` attribute defines the elements content based on the model object that it references.  

The `MODEL` attribute can reference either `<MODEL>` or `<FIELD>` data.  If it references a `<FIELD>`, then that field becomes the content of the element. In this example, the `<H1>` contains a reference to `article.headline`, and so the most recent `article.headline` becomes the content of `<H1>`.

If the `MODEL` attribute references a `<MODEL>`, then the element's attribute names are matched to `<FIELD>` names.  This is how to set an element's attributes.  A `<MODEL>` may also have an optional default field for the element's contents.

Since a model may contain multiple rows, which row would a model reference select?  By default, the most recent row.  Model references can also contain query selectors to select specific rows.  It is possible to have `<IMG model='image(id=3)'>` and this would always load the third image.  It is also possible to loop through lists of objects, as well as using `<TEMPLATE>` tags to instantiate new DOM objects.  Selectors can also be used to reference common error conditions.  More advanced link URLs could include SQL statements - `<A mref="http://...">` becomes `<A mref="sql:select from *">`.  Conventions would be defined for authentication.  The controller’s own `HREF` links could also be changed.  A link could update multiple model structures.  

Model data can be modified by Javascript if needed.  Model data could also be exported during form processing if desired. Anything in the DOM can be modified by `MODEL` attributes and/or Javascript, including `<HEAD>` elements, the URL bar, and style sheets.   External servers can also push data into models, using a Push API.  Models can also be connected directly to a local database for caching or persistence - the browser manages this now, instead of the web developer/Javascript.


Model Object
------------

A `<MODEL>` is equivalent to an SQL `TABLE`.  The `<FIELD>` elements would be the equivalent of an SQL `COLUMN`.  Fields are by default `TYPE=string`.  This would be the equivalent to an SQL `VARCHAR` field.  A `<MODEL>` element can optionally include a field, but it doesn't include a field by default.  More fields can be added via `<FIELD>` elements, as done in the examples here.  The `image` model in this example doesn’t have a default field, but several were added via `<FIELD>` elements.  All models should have at least one default and unique primary key - `ID` (or `index`).  It's not shown here, but it's possible to define foreign key references as well, via a `<REFERENCE>` element.  This example uses the `image` model as a child model of the `article` model.

Certain field types are used to define behavior. A `response` type can be used to indicate to the browser that this is going to be an API’s response, so let’s prepare for that.  Strings could have `maxLength`s, etc.  

The model data is versatile.  It can be used for various purposes, depending on the app's design and the API that it interfaces with.  The model data can be tied closely to the API server, leaving the client to perform much of the logic.  The model data can be tied closely to the client's view, leaving the API server to perform much of the logic.  The model data can be a temporary cache.  It may also be used offline.

The fun stuff starts to happen with these model definitions. This is where more advanced apps can go beyond simple CRUD.  It is possible to process these model objects in Javascript, and will be expanded on that here.  

Controller
----------

The web browser is the controller. 

Javascript event callbacks could be assigneed to model objects to perform actions.  There would be a set of events, similar to what the HTML presentation layer already has (`onMouseOver`, `onKeyPress`, etc.). For model objects, it would be things like `onLoad`, `onExpire`, etc.. - far too numerous to list here in this early concept stage.  It would include event message objects and bubbling, similar to Javascript’s event system.

The user interacts with the document view layer.  The user clicks on a link: `<A mref="http://api.mywebsite.com/article-2">`

The view layer notifies the browser of a click, and the browser fetches `http://api.mywebsite.com/article-2` and loads data into the model objects.  The model object then updates the document View layer.  All the logic to do that is in the browser.  

The browser already has a full Javascript interpreter that can be used to enhance this behaviour.  To optionally process models after loading, add a Javascript callback to the model's `onLoad` method.

This maintains the separation of concerns in HTML.  HTML isn’t doing logic here.  The browser is still responsible for logic, and it has default behaviors for these models ("Clicking on this link loads external data into this model") which can be extended ("convert the Markdown model data into HTML", or "load API data at a scroll point")

For example, to translate JSON Markdown text object into HTML, attach a callback function to `onLoad`:

	<MODEL name="article" onLoad="MarkdownToHTML">

This callback could be attached with Javascript:

	models.article.body.onLoad=function(){MarkdownToHTML};

or:

	models.article.body.addEventListener("onLoad", MarkdownToHTML)

Model objects can be accessed via a Javascript API:

	models.article.get("id=1").headline = "Dewey Defeats Truman"

or directly via arrays on the primary key:

	models.article[1].headline = "Nope"

The above Markdown converter could be applied to just the article `body`:

	models.article.body = function(){MarkdownToHTML};

Model objects can subscribe to a push API server:

	models.article.pushSubscribe("http://api.mynewsserver.com/user-id-5678")

or:

	<MODEL name="article" src="http://api.mynewsserver.com/user-id-5678">

And whenever a new message is received, a function can be applied:

	models.article.onPush=function(){UpdateMessageCount}

Forms
-----

Form fields interact with models directly:

	<INPUT type="text" model="article.headline">

`<FORM>` elements are unnecessary for this interaction with the model, as it updates live. `<FORM>` elements are required to send the data to the server via a `<BUTTON>`, and `<FIELD>` elements can be used to send model data:

	<FORM action="http://api.mynewsserver.com/edit-article">
		<FIELD model="myAppInfo.api_key">
		<FIELD model="article.id">
		<FIELD model="article.headline">
		<FIELD model="article.body">
		<BUTTON type="submit">Edit Article</BUTTON>
	</FORM>

To receive data into a model using a form:

	<FORM action="http://api.mynewsserver.com/load-article" receiver="article">

With this, a basic but highly-responsive single-page CRUD app can be made without Javascript.

Further Uses
------------

For more custom procedures in Javascript, there would be an equivalent to `XMLHttpRequest` API for model interactions:

	var request = new ModelHttpRequest();
	request.open(‘GET’, 'http://api.mynewsserver.com/get-latest-article', true);
	request.receiverModel(myArticleData)
	request.send()

To send a model data via Javascript instead of an `<FORM>`:

	var request = new ModelHttpRequest();
	request.open(‘POST’, 'http://api.mynewsserver.com/edit-article', true);
	request.sendModelData([MyAppData.api_key, myArticleData])
	request.send()

To always send new objects whenever a model is updated:

	<MODEL name="article" onChange="mySendFunction">

It is also possible to define custom non-default response handlers at this point:

	<MODEL name="article" onResponse="myResponseHandler">

For presentation, the `<TEMPLATE>` element can be used to show model objects in lists:

	<UL class="headLines">
		<TEMPLATE model="article">
			<LI model="this.headline"></LI>
		</TEMPLATE>
	</UL>

This would be the equivalent to "`block`" structures in other templating languages, with `this` accessors available within a template block.  

The following is a `<TEMPLATE>` example with infinite scroll:

	<UL class="headLines">
		<TEMPLATE model="article">
			<LI model="this.headline" onShow="loadMoreArticles"></LI>
		</TEMPLATE>
		<TEMPLATE model="article">
			<LI model="this.headline"></LI>
		</TEMPLATE>
		<TEMPLATE model="article">
			<LI model="this.headline"></LI>
		</TEMPLATE>
	</UL>

Each reference to the same model cycles through model objects here.

Advanced users may use the object store for caching purposes:

	<MODEL name="article" src="http://api.mynewsserver.com/user-id-5678" expire=300>

This would invalidate records after 300 seconds.  The `SRC` connection could also push a cache invalidation. 

Caching is a bit complicated and in order to keep this high-concept design note quick & simple, we will leave it for now.

SQL Queries
-----------

Data can be accessed from a server via SQL Queries. This data can then be used to populate the model objects.

This is designed with a front-end client sandbox perspective, not a back-end server perspective that databases typically operate in.  There are important security issues, but they matters less when operating on local data for a local app, like a single-player game on a mobile device.

There are also millions of non-interactive websites with static content, such as portfolio or business-card sites, that also don't need to worry about complex security requirements.  For these sites, database writes should be disallowed by default.

For more complex multi-user websites on the Internet, a possible use model would be for the server to transform local SQL statements into something that applies to the server schema.  The proposal here makes no assumption about the server architecture and data that’s being exposed by the server for the client SQL syntax.  The server is still responsible for securing its data, as always.

A server that receives the following command:

	SELECT first_name, last_name FROM users;

May parse that and translate that into something usable like:
	
	SELECT first_name, last_name FROM users WHERE manager="Boss Man";

But, if it receives the following SQL command:

	DROP TABLE students;

A simple server framework might not even be able to parse it and may just ignores it, while a more advanced server framework might try to authenticate and proceed with it.  Meanwhile, for a local SQL database, like for a game on your mobile device, it might very well delete the table.  But for a remote server, anything could happen, depending on the server’s design.  

When the server does this type of transform before hitting the database, it basically offers the client a sandboxed subset of the database schema.  The client might not even see nor be aware of the `manager` column here, for example.  The front-end user just sees this sandbox, and happily operates only with that data.  Since it’s up the the server to do this type of transform, the server might not even be using an SQL RDBMS, and it might transform or interface into something else entirely.  This is also where the server might protect against SQL injection attacks, if using an SQL RDMBS.

So, for an Internet client-server setup, there would likely be an app server in between the client and the database, like how things are now, and this proposal offers a different syntax.

Right now, if an app server gets this request:

	http://api.mywebsite.com/get_article?id=1234

It may happily oblige.

But if it receives this:

	http://api.mywebsite.com/delete_article?id=1234	
    
It would have to authenticate.  Server-side app designers still have to implement their security checks, like they would in a framework like Django:

	from django.contrib.auth import authenticate
	user = authenticate(username='john', password='secret')
	if user is not None:
   		# the password verified for the user
   		if user.is_active:
       		print("User is valid, active and authenticated")
		    else:
		        print("The password is valid, but the account has been disabled!")
	else:
  		 # the authentication system was unable to verify the username and password
	 	print("The username and password were incorrect.”)

A server-side designer still has to write this kind of code.  This proposal doesn’t remove that requirement.

This proposal means to put the idea of using standard SQL syntax to get/set data for structured remote API data access.  As this proposal is further built, there would be an ORM that could be standardized so that clients could use this SQL syntax.  

Finally, this SQL interface would be optional.  On the internet, you would most likely be using public JSON APIs at first anyways.  The SQL interface was designed to eliminate a layer of back-end when using a local database app, as well as eliminating the need to use Javascript for such local database accesses.  



