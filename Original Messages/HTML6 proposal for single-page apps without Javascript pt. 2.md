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
