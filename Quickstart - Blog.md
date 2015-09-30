Quickstart: Blog
----------------

There are three initial examples in this Quickstart.

 -

1. Static HTML5 page: The "before" picture
------------------------------------------

The first example depicts what would be a tradional static HTML5 article page.  It's placed here as an example of the "before" picture. 

In this example, links in the table-of-contents menubar are used to guide the reader to different article pages on the site.  When a reader clicks on a link in their browser, the full article page is loaded.


    <!DOCTYPE html>
    <HTML lang="en">
    <HEAD>
        <TITLE>My blog post.</TITLE>
    </HEAD>
    <BODY>
        <MENU>
            <UL>
                <LI><A href="article-1.html">Click to read Article 1.</A></LI>
                <LI><A href="article-2.html">Click to read Article 2.</A></LI>
            </UL>
        </MENU>
        <MAIN>
            <ARTICLE>
                <IMG SRC="image.jpg">
                <H1>Big News!</H1>
                <SPAN><P>This is the initial article.</P></SPAN>
            </ARTICLE>
        </MAIN>
    </BODY>
    </HTML>


2. Introducing <MODEL> elements for articles
--------------------------------------------

For our first steps into a model-driven web, we will create <MODEL> elements in the <HEAD>.  These model elements will be used to populate the tags in the article body.


    <!DOCTYPE html>
    <HTML lang="en">
    <HEAD>
        <MODEL name="article">
            <FIELD name="headline">
            <FIELD name="body">
            <FIELD name="image" type="IMG">
        </MODEL>
    </HEAD>
    <BODY>
        <MENU>
            <UL>
                <LI><A href="article-1.html" mref="http://api.mywebsite.com/article-1" receiver="article">Click to read Article 1.</A></LI>
                <LI><A href="article-2.html" mref="http://api.mywebsite.com/article-2" receiver="article">Click to read Article 2.</A></LI>
            </UL>
        </MENU>
        <MAIN>
            <ARTICLE>
                <IMG model="article.image">
                <H1 model="article.headline">Big News!</H1>
                <SPAN model="article.body"><P>This is the initial article.</P></SPAN>
            </ARTICLE>
        </MAIN>
    </BODY>
    </HTML>


By default each <FIELD> element is of type "text".  When a model is assigned to an element, such as <H1 model="article.headline"></H1>, the content of the element is replaced with the text.

There is a built-in "IMG" field type, and that allows models to include attribute information, such as image WIDTH and HEIGHT.  

In fact, there are built-in types for every single HTML Element.  (<FIELD TYPE="DIV">, <FIELD TYPE="P">, etc.. ) This allows models to include the various attributes of every Element type, including global attributes.  These attributes are now accessible as part of the model definition.  Finally, when a model is assigned to an element, these attributes are attached to the element.


3. Using <MODEL> elements for the Table-of-Contents
---------------------------------------------------

This example shows you how to build a Table-of-contents menubar using the <MODEL> element.

 
    <!DOCTYPE html>
    <HTML lang="en">
    <HEAD>
        <MODEL name="article">
            <FIELD name="headline">
            <FIELD name="body">
            <FIELD name="image" type="IMG">
        </MODEL>
        <MODEL name="contents">
            <FIELD name="link" type="A">
        </MODEL>
    </HEAD>
    <BODY>
        <MENU>
            <UL>
                <LI><A model="contents" row="1">Click to read Article 1.</A></LI>
                <LI><A model="contents" row="2">Click to read Article 2.</A></LI>
                <LI><A model="contents" row="3">Click to read Article 3.</A></LI>
            </UL>
            <SPAN><A model="contents">Older Articles.</A></SPAN>
            <SPAN><A model="contents">Newer Articles.</A></SPAN>            
        </MENU>
        <MAIN>
            <ARTICLE>
                <IMG model="article.image">
                <H1 model="article.headline">Big News!</H1>
                <SPAN model="article.body"><P>This is the initial article.</P></SPAN>
            </ARTICLE>
        </MAIN>
    </BODY>
    </HTML>


    <MODEL name="article">
        <FIELD name="headline">
        <FIELD name="body">
        <FIELD name="image" type="image">
    </MODEL>
    <MODEL name="map">
        <FIELD name="temperature" type="tempertature">
        <REFERENCE name="state"">
    </MODEL>





