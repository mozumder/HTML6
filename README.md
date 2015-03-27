_This document is volatile & updated several times daily.  Check back often!  And feedback is appreciated._

Problem
-------

There’s a standard design pattern emerging in large front-end Javascript frameworks (Angular, Ember, React, etc.) where content is loaded dynamically through JSON APIs and displayed on the page.  This is commonly known as the single-page web app.  

Web developers do this to help the user's web browsing experience.  Normally, whenever a user clicks on a link, the browser loads a completely new HTML document.  Meanwhile, on a typical website, the pages share a common HTML document structure.  As a user traverses a site, the server has to generate the same parts of the page over and over, and the browser has to reload that same data over and over.  This wastes time and power, and can result in a noticeable lag when interacting with a website.  So, web developers use these Javascript frameworks to cut down on that lag.  When a user clicks a link, these Javascript frameworks only load certain parts of the HTML document.  Since the document structure typically doesn't change for a site, they can just reload the elements that do change. Doing this shaves several milliseconds when interacting with a webpage, which can be a critical to the user's experience.

Meanwhile, there are several obvious problems with this approach.  Every web page now has to include a huge Javascript framework that the browser has to load and compile.  These frameworks can be several hundred KB for a page.  This takes time and power.  These Javascript frameworks aren't easy to use, and have different features and capabilities.   They’re all trying to figure it out and in a state of flux. Angular is being redesigned and will break backwards compatibility for 2.0.  Ember is also being heavily upgraded.  Backbone & React doesn’t do the full flow, as you still need to bring in other frameworks for things like persistent data modeling.  There are probably new frameworks popping up weekly as well.

Additionally, a big problem is Javascript itself.  Should we force millions of web developers to learn Javascript, a framework, and an associated templating language, if they want a speedy and responsive website out-of-the-box?  This is a huge barrier for beginners, so right now they’re stuck with slower full page reloads, because by default, HTML loads full web pages.

So there's a real, heavy cost associated with these frameworks.  This cost needs to be reduced for a successful web.


Proposal
--------

Since this web design pattern is so common now, we would like to see browsers implement this directly via enhancements to HTML so that users can dynamically run single-page apps without having to use Javascript.

This could be done by declaring model objects in the HTML `<HEAD>`:

    <MODEL class="myArticleData">

`<A>` elements would specify JSON/XML API endpoints and models that receive data:

    <A mref="http://api.mywebsite.com/get-article" receiver="myArticleData">

Finally, the DOM is dynamically updated through model references:

    <ARTICLE model="myArticleData">

Initial data, as well as standard error responses, could be in body elements as well as header fixtures, which may be replaced later.  

Thus HTML becomes a templating language, with content residing in model objects that can be dynamically reloaded without Javascript.

Example
-------

    <DOCTYPE html>
    <HTML LANG="en">
    <HEAD>
    <MODEL class="myArticleData">
        <MODEL class="rsp" type="response">
            <MODEL class="article">
                <PROPERTY name="label" type="string">
                <MODEL class="headline" type="string">  
                <MODEL class="body" type="string">
            </MODEL>
        </MODEL>
    </MODEL>
    <MODEL class="myImageData">
        <MODEL class="rsp" type="response">
            <MODEL class="image">
                <PROPERTY name="label" type="string">
                <PROPERTY name="width" type="integer">
                <PROPERTY name="height" type="integer">
                <PROPERTY name="source" type="url">
            </MODEL>
            <FIXTURE lang="json">
                {
                   "stat": "ok",
                   "image": [
                      {
                         "id": "3",
                         "label": "Square",
                         "width": "75",
                         "height": "75",
                         "source": "https://mycontentserver.com/image_s.jpg",
                      },
                      {
                         "id": "4",
                         "label": "Tall",
                         "width": "300",
                         "height": "200",
                         "source": "https://mycontentserver.com/image_l.jpg"
                      }
                   ]
                }
            </FIXTURE>
            <FIXTURE lang="json">
                {
                   "stat": "loading",
                   "image": {
                      "id": "1",
                      "label": "Square",
                      "width": "75",
                      "height": "75",
                      "source": "https://mycontentserver.com/loading_image_s.jpg"
                   }
                }
            </FIXTURE>
            <FIXTURE lang="json">
                {
                   "stat": "some_error",
                   "image": {
                      "id": "2",
                      "label": "Square",
                      "width": "75",
                      "height": "75",
                      "source": "https://mycontentserver.com/error_image_s.jpg"
                   }
                }
            </FIXTURE>
        </MODEL>
    </MODEL>
    </HEAD>
    <BODY>
        <MENU class="controller">
            <A href="article-2.html" mref="http://api.mywebsite.com/article-2" receiver="myArticleData">Click here to replace the articles with different articles.</A>
            <A href="image-2.html" mref="http://api.mywebsite.com/image-2" receiver="myImageData">Click here to replace the picture with a different picture.</A>
        </MENU>
        <MAIN class="viewer">
            <ARTICLE class="center">
                <H1 model="MyArticleData.rsp.article(label='one').headline">Big News!</H1>
                <SPAN model="MyArticleData.rsp.article(label='one').body"><P>This is the first article.</P></SPAN>
            </ARTICLE>
            <ARTICLE class="sidebar">
                <H1 model="MyArticleData.rsp.article(label='two').headline">Not so big news</H1>
                <SPAN model="MyArticleData.rsp.article(label='two').body"><P>This is the second article.</P></SPAN>
            </ARTICLE>
            <IMG src="model:MyImageData.rsp.image(label=‘Square’)#source" width="model:MyImageData.rsp.image(label=‘Square’)#width" height="model:MyImageData.rsp.image(label=’Square’)#height">
        </MAIN>
    </BODY>
    </HTML>


Clicking on a link loads JSON data from the API endpoint specified in `MREF` property into a destination data model object specified in the `RECEIVER` property.  This is destination model object is separate from the DOM.   The `HREF` property remains as a canonical URL for the page, and can be used by older browsers that don't support dynamic page updates.  Older browsers load the canonical `HREF` URL.

The initial fixture data can be loaded from the `<FIXTURE>` element in a `<MODEL>`, but these fixtures can be loaded via an external link, or implicitly by default from within `<BODY>` elements that contain model references. In this example, the `myArticleData` model loads model fixtures from the `<H1>` and `<SPAN>` elements, while the `myImageData` model loads in fixtures from the `<FIXTURE>` elements.  The overall schema can be defined explicitly by `<MODEL>` elements, or implicitly by `<FIXTURE>`, elements, or even through SQL statements.

The text sections have `<H1>` and `<SPAN>` tags with a new `MODEL` attribute that defines its content based on the model object data.  This format is declarative, but can approach SQL's' complexity.  You don’t need presentation-layer controller/view structures, but this example includes them.

There is also a `model:` URI that old attributes and the rest of the page can use to references the loaded data.  Anything in the DOM should be replaceable, including the URL bar and style sheets.  This internal data can be modified by Javascript if needed, separate from modifying the DOM - you could export this data for form processing if you wish.  This internal data can also be push updated by a server or connected directly to a local database for caching or persistence - the browser manages this now, instead of the web developer/Javascript.

Model references can call any object, with the ability to define conditions to access objects, perhaps looping through lists of objects and using `<TEMPLATE>` tags to instantiate new DOM objects.  There should be standard conventions for common error control.  More advanced link URLs could include SQL statements - `<A mref="http://...">` becomes `<A mref="sql:select from *">`.  Conventions would be defined for authentication.  The controller’s own `HREF` links could also be changed.  A link could update multiple model structures.  

Model Object
------------

A `<MODEL>` is equivalent to an SQL TABLE.  Both the model’s `TYPE` attribute and `<PROPERTY>` elements would be the equivalent of an SQL `COLUMN`.  For example, the `article` model has a `TYPE="string"`, and this means the article model has a default field that is equivalent of an SQL `VARCHAR` field.  We can add more fields via `<PROPERTY>` elements, as done in the example.  The `image` model in this example doesn’t have a default field, but we added several via `<PROPERTY>` elements.  All models should have at least one default and unique primary key - ID.  I don’t show it here, but this is where we would have foreign key references as well, via a `<REFERENCE>` elements.  This example uses child model definitions to make it clear.

Certain model types are used to define behavior. The `response` type for the `rsp` model indicates to the browser that this is going to be an API’s response, so let’s prepare for that.  Strings could have `maxLengths`, etc.

The fun stuff starts to happen with these model definitions. This is where you would have more advanced apps that go beyond simple CRUD.  You could process these objects in Javascript if you wish, and will expand on that here.  

Controller
----------

The web browser is the controller. 

We could assign Javascript event callbacks to these models or properties to perform actions.  There would be a whole set of events, similar to what the HTML presentation layer already has (`onMouseOver`, `onKeyPress`, etc.). For model objects, it would be things like `onLoad`, `onExpire`, etc.. - far too numerous to list here in this early concept stage.  It would include event message objects and bubbling, like the rest of Javascript’s event system.

The user interacts with the document view layer.  The user clicks on a link: '<A mref="http://api.mywebsite.com/article-2">'

The view layer notifies the browser of a click, and by default the browser fetches 'http://api.mywebsite.com/article-2' and loads data into the model objects.  The model object then by default updates the document View layer.  All the logic to do that is in the browser.  

The browser already has a whole Javascript framework that can be used to enhance its behaviour.  If you want to do some processing on the models, just add a Javascript callback to the model's `onLoad` method.

This maintains the separation of concerns in HTML.  HTML isn’t doing logic here.  The browser is still responsible for logic, and it has default behaviors for these models ("Clicking on this link loads external data into this model") which you can extend ("convert the Markdown model data into HTML", or "load API data at a scroll point")

For example, if you wanted to translate a JSON Markdown text object into HTML for presentation, you could do that in a callback function attached in the HTML:

	<MODEL class="MyArticleData" onLoad="MarkdownToHTML">

They could be attached in Javascript:

	models.MyArticleData.onLoad=function(){MarkdownToHTML};

or:

	models.MyArticleData.addEventListener("onLoad", MarkdownToHTML)

You could access model objects via a Javascript API:

	models.MyArticleData.rsp.get("stat=‘ok’").article.get("id=1").headline = "Dewey Defeats Truman"

or directly via arrays on the primary key:

	models.MyArticleData.article[1].headline = "Nope"

(I collapsed in the `rsp` model here into `MyArticleData` as a feature of the `response` model type.)

You could apply the above Markdown converter to just the article `body` model object:

	models.MyArticleData.article.body = function(){MarkdownToHTML};

You could do things like subscribe to a push API server:

	models.MyArticleData.pushSubscribe("http://api.mynewsserver.com/user-id-5678")

or:

	<MODEL class="MyArticleData" src="http://api.mynewsserver.com/user-id-5678">

And whenever you get a new message, you can update the page:

	models.MyArticleData.onPush=function(){UpdateMessageWindow}

Forms
-----

Form fields interact with models directly:

	<INPUT type="text" model="MyArticleData.article.headline">

You don’t need a `<FORM>` element for this interaction with the model, as it updates live. You do need it if you want to send the data to the server via a `<BUTTON>`, and we could add a `<FIELD>` element for sending model data:

	<FORM action="http://api.mynewsserver.com/edit-article">
		<FIELD model="MyAppData.api_key">
		<FIELD model="MyArticleData.article.id">
		<FIELD model="MyArticleData.article.headline">
		<FIELD model="MyArticleData.article.body">
		<BUTTON type="submit" >Edit Article</BUTTON>
	</FORM>

If you wanted to receive data into a model as well:

	<FORM action="http://api.mynewsserver.com/load-article" model="MyArticleData">

With this, you could create basic but highly-responsive single-page CRUD apps without Javascript.

Further Uses
------------

For more custom stuff in Javascript, there would be an equivalent to `XMLHttpRequest` API, except for model interactions:

	var request = new ModelHttpRequest();
	request.open(‘GET’, 'http://api.mynewsserver.com/get-latest-article', true);
	request.receiverModel(MyArticleData)
	request.send()

If you wanted to send a model via Javascript instead of an HTML `<FORM>`:

	var request = new ModelHttpRequest();
	request.open(‘POST’, 'http://api.mynewsserver.com/edit-article', true);
	request.sendModelData([MyAppData.api_key, MyArticleData])
	request.send()

There would be lots of basic capabilities here, such as translating API names to model names.

If you always wanted to send new objects whenever a model is updated:

	<MODEL class="MyArticleData" onChange="mySendFunction">

This is also where you could define custom non-default response handlers if you wish:

	<MODEL class="MyArticleData" onResponse="myResponseHandler">

As for presentation, use the `<TEMPLATE>` element to show model objects in lists:

	<UL class="headLines">
		<TEMPLATE model="MyArticleData.article">
			<LI model="this.headline"></LI>
		</TEMPLATE>
	</UL>

This would be the equivalent to "`block`" structures in other templating languages, with `this` accessors available within a template block.  

Maybe you want infinite scroll:

	<UL class="headLines">
		<TEMPLATE model="MyArticleData.article">
			<LI model="this.headline" onShow="loadMoreArticles"></LI>
		</TEMPLATE>
		<TEMPLATE model="MyArticleData.article">
			<LI model="this.headline"></LI>
		</TEMPLATE>
		<TEMPLATE model="MyArticleData.article">
			<LI model="this.headline"></LI>
		</TEMPLATE>
	</UL>

Each reference to the same model cycles through model objects here.

Advanced users might use the object store for caching purposes:

	<MODEL class="MyArticleData" src="http://api.mynewsserver.com/user-id-5678" expire=300>

This would invalidate records after 300 seconds.  The `SRC` connection could also push a cache invalidation. 

Caching is a bit complicated and in order to keep this high-concept design note quick & simple, we will leave it for now.

SQL Queries
-----------

Data can be accessed from a server via SQL Queries. This data can then be used to populate the model objects.

This is designed with a front-end client sandbox perspective, not a back-end server perspective that databases typically operate in.  There are important security issues, but they matters less when operating on local data for a local app, like a single-player game on a mobile device.

There are also millions of non-interactive websites with static content, like portfolio or business-card sites, that also don't need to worry about complex security requirements.  For these sites, database writes should be disallowed by default.

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




