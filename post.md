Data Denormalization: Going from relational to Orchestrate
====
You may have just moved from Postgres to Orchestrate, and found your application doing several queries from multiple collections to load a single page. You may have your data organized neatly in different collections. 

The higher performance of Node.JS over Python has taught us that clean looking code does not relate to higher performance. The neat relational-style layout may be causing a performance problem.

Same data, different layouts
---
You can layout your data in many different ways. In a blog engine, you could have a collection for users, a collection for posts, and a collection for comments. Or, you could have the comments in a subdocument with the user who posted it in the collection for 'users', leaving you with two collections. You could also use the 'user' collection for everything, where the user's documents contain all comments and articles as subdocuments. When should you use a separate collection for a separate type of data versus keeping it inside of a larger JSON document?

Keeping data in their own separate collection (or tables) when the data does not relate one-to-one is called normalizing. If every use must have one and only one username and email, then the password may go into the same JSON document (or table row) as the username in a normalized database. If the user can have more than one comment, the comments must go into the separate collection from the username and email. Traditionally, relational databases have a normalized layout.
With orchestrate, a normalized layout of this could look like
```javascript
{
	"username": "tony",
	"email": "tony@orchestrate.io"
}
```
with
```javascript
{
	"by": "tony",
	"articleid": "denormalization",
	"content": "First Post!!",
	"date": Date()
}
```

Why Denormalize?
---
The opposite of normal
![alt text](https://raw.githubusercontent.com/evanroman/orchestrateblogpost/master/1024px-Keep_Portland_Weird.jpg "The opposite of normal")
In a normalized database, retrieving data from multiple collection, like loading a user's profile that has their comments and their contact info, two queries must be made: one query for each collection
```bash
curl -i "https://api.orchestrate.io/v0/users/tony" \
    -u "$your_api_key:"
curl -i "https://api.orchestrate.io/v0/comments?query=user:tony&offset=0" \
    -u "$your_api_key:"
```
Having more queries increases latency.
Unlike relational databases, Orchestrate can store data in subdocuments within documents. To reduce the number queries for this scenario, placing the comments into the `user` collection like so only requires one query to load the user's profile
```javascript
{
	"username": "tony",
	"email": "tony@orchestrate.io",
	"comments": {
		"denormalization": {
			"date": Date(),
			"content": "First Post!!"
		}
	}
}
```

Why Normalize
---

Keeping data separate does make the data more organized and easier to work with. If the two type of data will not be simultaneously queried, then keeping them in the same collection does not offer any performance benefit.

If you are migrating from a relational database like MySQL or Postgres, figure out how you will be querying your data without `JOIN` operations. You need to know what ways your application will query your data, and if keeping 1:n table relations makes sense for how your app will query your data. With properly planned queries, almost all relational data can fit into the scalable and accessible Orchestrate. 
