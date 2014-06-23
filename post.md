Data Denormalization: Going from relational to Orchestrate
====
You may have just moved from Postgres to Orchestrate, and found your application doing several queries from multiple collections to load a single page. You may have your data organized neatly in different collections. 

The higher performance of Node.JS over Python has taught us that clean looking code does not relate to higher performance. The neat relational-style layout may be causing a performance problem.

Same data, different layouts
---
You can layout your data in many different ways. In a blog engine, you could have a collection for users, a collection for posts, and a collection for comments. Or, you could have the comments in a subdocument with the user who posted it in the collection for 'users', leaving you with two collections. You could also use the 'user' collection for everything, where the user's documents contain all comments and articles as subdocuments. When should you use a separate collection for a separate type of data versus keeping it inside of a larger JSON document?

Keeping data in their own separate collection (or tables) when the data does not relate one-to-one is called normalizing. If every use must have one and only one username and email, then the password may go into the same JSON document (or table row) as the username in a normalized database. If the user can have more than one comment, the comments must go into the separate collection from the username and email. Traditionally, relational databases have a normalized layout.
With orchestrate, a normalized layout of this could look like:
`users` profiles with `username` as their respective keys:
```javascript
{
	"username": "tony",
	"email": "tony@orchestrate.io",
	"name": "Tony Falco"
}
{
	"username": "evan"
	"email":"evanroman1@gmail.com",
	"name": "Evan Roman"
}
```
`comments`, with a uuid as their keys:
```javascript
{
	"by": "tony",
	"articleid": "denormalization",
	"content": "First Post!!",
	"date": 1403507059218
}
{
	"by": "tony",
	"articleid": "denormalization",
	"content": "Second post.",
	"date": 1403507082905
}
```
and `articles` with `articleid` as their respective keys:
```javascript
{
	"by": "evan"
	"title": "Data Denormalization: Going from relational to Orchestrate",
	"articleid": "denormalization",
	"content": "super meta",
	"date": 1403507042197
}
```
Why Denormalize?
---
The opposite of normal
![alt text](https://raw.githubusercontent.com/evanroman/orchestrateblogpost/master/1024px-Keep_Portland_Weird.jpg "The opposite of normal")
In a normalized database, retrieving data from multiple collection, like loading a user's profile that has their comments and their contact info, two queries must be made: one query for each collection. Having more queries increases latency. Unlike relational databases, Orchestrate can store data in subdocuments within documents. To reduce the number queries for this scenario, placing the comments into the `users` collection like so only requires one "Get" query to `users` with the key "tony" will load the user's profile and their comments. 
```javascript
{
	"username": "tony",
	"email": "tony@orchestrate.io",
	"comments":
		[
			{
				"articleid": "denormalization",
	   	   		"date": 1403507059218,
   	   			"content": "First Post!!"
			},
			{
				"articleid":"denormalization",
		   		"date": 1403507082905,
	   			"content":"Second post."
   			}
		]
	}
}
```

More blog views go to the articles. To view a page with the article and the comments, a query of `users` for their comments and the seperate collection `articles` must be made: one "Get" to `articles` with the key "$articleid," and another to `users` with the search query 'articleid:"$articleid"'. The search of the `users` database will reply with the entire documents of all users who have comments on the particular article, so the application server will recieve these user's comments for all articles and their user profile information. The larger amount of information sent increases bandwith and requires your application to sort through a larger amount of JSON data than it needs to too. If the `users` collection stores the article data as subdocuments in their author's `user` documents, then only one query would be needed to load the article and comments, but the amount of unneeded JSON data for each query will grow. This may improve performance overall, but it puts a heavier load on your application server, and requires more complex application logic to search through the recieved JSON.

If the `articles` collection stored the comments, then all article reads would require one "Get" query and return only the JSON needed for that specific article. This also means that two copies of the same comments data would be stored in seperate collections. At the cost of higher storage, all pages can be loaded with one query that respond without extraneous information. In this model, every type of page type would have their own collection.
For this blog, having a collection for each page could like:
`homepage` lists the articles for the homepage of the blog:
```javascript
{
	"articleid":"denormalization",
	"title": "Data Denormalization: Going from relational to Orchestrate",
	"date": 1403507042197,
	"summary":"You may have just moved from Postgres to Orchestrate, and found your application doing several queries from multiple collections to load a single page. You may have your data organized neatly in different..."
}
```
`users` would look the same as the denormalized version shown further above.
`articles` includes all the needed information for showing an article with comments:
```javascript
{
	"articleid":"denormalization",
	"title": "Data Denormalization: Going from relational to Orchestrate",
	"date": 1403507042197,
	"content":"super meta",
	"comments":
		[
			{
				"user": "tony",
	   	   		"date": 1403507059218,
   	   			"content": "First Post!!"
			},
			{
				"user":"tony",
		   		"date": 1403507082905,
	   			"content":"Second post."
   			}
		]
}
```
Picking the best way
---

Keeping data separate does make the data more organized and easier to work with. If the two types of data will not be simultaneously queried, then keeping them in the same collection does not offer any performance benefit. The database may reply with some extraneous data, but far less than Denormalizing without redundancy.

Denormalizing without redundancy will allow fast, single query reads for data optimized for reads, while data unoptimized for reads will require more than one query and possibly large amounts of extraneous data replied from the database. For example, having comments exclusively in the user's document, will make reading a user profile page with their comments fast, while loading an article with comments would require multiple queries and the application would recieve extraneous information (the entire user document with all comments for all articles, for ever user who commented on this article). To optimize for reading articles, the comments would be stored with the articles.

Denormalizing with redundancy will allow fast, single query reads for all queries. Every possible query your application makes has its own collection. All queries will be free of extraneous data. This will use more storage space, since the same information is stored in multiple places, and require the application to write to multiple collections when one piece of redundant data is changed.

These different methods can be used together. If you are migrating from a relational database like MySQL or Postgres, figure out how you will be querying your data without `JOIN` operations. You need to know what ways your application will query your data, and if keeping 1:n table/collection relations with shared keys makes sense for how your app will query your data in the abscence of `JOIN` operations. With queries planned in advance, almost all relational data can fit into and be used efficiently by the scalable and accessible Orchestrate. 
