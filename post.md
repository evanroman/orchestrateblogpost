Data Denormalization: Going from relational to orchestrate
====

You can layout your data in many different ways. In a blog engine, you could have a collection for users, a collection for posts, and a collection for comments. Or, you could have the comments in a subdocument with the user who posted it in the collection for 'users', leaving you with two collections. You could also use the 'user' collection for everything, where the user's documents contain all comments and articles as subdocuments. When should you use a seperate collection for a seperate type of data versus keeping it inside of a larger JSON document?

Keeping data in their own seperate collection (or tables) when the data does not relate one-to-one is called normalizing. If every use must have one and only one username and email, then the password may go into the same JSON document (or table row) as the username in a normalized database. If the user can have more than one comment, the comments must go into the seperate collection from the username and email. Traditionally, relational databases have a normalized layout.
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
	"email: "tony@orchestrate.io",
	"comments": {
		"denormalization": {
			"date": Date(),
			"conent": "First Post!!"
		}
	}
}
```

If you are migrating from a relational database like MySQL or Postgres, figure out how you will be querying your data with `JOIN` operations. You need to know what ways your application will query your data, and if keeping 1:n table relations makes sense for how your app will query your data. With properly planned queries, almost all relational data can fit into the scalable and acessable Orchestrate. 
