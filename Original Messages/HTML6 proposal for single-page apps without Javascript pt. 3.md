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

