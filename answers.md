**Question**: How does GlideQuery work? 

**Answer**: GlideQuery is a JavaScript API provided by ServiceNow. It is used for querying database tables in a more modern and intuitive way compared to the traditional GlideRecord and GlideAggregate APIs. 

At a basic level, you can think of GlideQuery as an object-oriented way to build and execute database queries. You start by creating an instance of GlideQuery for a specific table, and then you chain together various methods to build up your query. Once your query is built, you execute it to retrieve the results.

Here's a simple example:

```javascript
var users = new GlideQuery('sys_user')
  .where('active', true)
  .select();
```

In this example, we're creating a new GlideQuery for the `sys_user` table, adding a WHERE clause to only include active users, and then executing the query with `select()`. The result is an array of objects, each representing a user record.

Under the hood, GlideQuery is creating a SQL-like query based on the methods you chain together. It uses the `table` property to know which table to query. It uses the `plan` property, which is an array of "step" objects, to keep track of the query operations (like SELECT, WHERE, ORDER BY, etc.). Each step object describes a part of the query.

When you execute the query, it uses the `GlideQueryEvaluator` to turn the plan into an actual query that can be executed against the database. It also checks the schema of the table to make sure the fields you're querying actually exist.

Note: As of now, GlideQuery is only available in scoped applications and starting from the 'Orlando' release. 

**Question**: How is GlideQuery implemented that it allows for you to chain methods on top of each other? 

**Answer**: GlideQuery, a ServiceNow class, allows for method chaining in its implementation. Method chaining is a common programming technique in which multiple methods are called sequentially on an object, with each method returning an object that can support further methods. This allows for the creation of a 'chain' of method calls. 

In GlideQuery, each method like `.select()`, `.orderBy()`, `.where()`, etc., returns a new instance of the GlideQuery object. This allows the methods to be chained together. 

For instance, you can have something like this:

```javascript
var users = new GlideQuery('sys_user')
    .select('first_name')
    .where('last_name', '=', 'Smith')
    .orderBy('first_name')
    .toArray();
```

This is an example of method chaining in the GlideQuery object. Here, `.select('first_name')` returns a GlideQuery object that has the select method applied, `.where('last_name', '=', 'Smith')` returns a new GlideQuery object that has the where method applied, and so on. This allows for the methods to be chained together. 

The key to this chaining is that each of these methods returns a new GlideQuery object with the current state. This is evident from the code snippet:

```javascript
GlideQuery.prototype.withAcls = function withAcls() {
	return new GlideQuery(
		this.table,
		this.plan.concat({
			type: 'withAcls',
		})
	);
};
```

In the `withAcls()` method, after applying the operation, a new GlideQuery object is returned with the current state. This allows for subsequent method calls to be made on the same line. 

This technique of method chaining makes the code cleaner and more readable. 

**Question**: What about the toArray() method, I don't understand why we need that. What does it do? 

**Answer**: The `toArray()` method in GlideQuery is a terminal function that executes the query and returns the results as an array of simple JavaScript objects. Each object in the array represents a record from the database and the properties of the object are the fields of the record. 

This method is useful when you need to work with the result set in a format that's easy to manipulate with standard JavaScript. It allows you to iterate over the result set using array methods like `forEach()`, `map()`, `filter()`, etc.

Here's a simplified example:

```javascript
var users = new GlideQuery('sys_user')
    .select('name', 'email')
    .where('active', true)
    .limit(10)
    .toArray();

// Now users is an array of objects, and we can manipulate it with standard array methods
users.forEach(function(user) {
    console.log(user.name + ': ' + user.email);
});
```

In this example, `users` is an array of objects where each object has a `name` and `email` property. We can then easily loop through the array and do something with each user.

It's important to remember that the `toArray()` method executes the query, so any modifications to the GlideQuery after calling `toArray()` won't affect the result set. 

**Question**: How can I use regex with GlideQuery? 

**Answer**: In ServiceNow, regular expressions can be used with GlideQuery in a number of ways to perform pattern matching and find specific sequences in strings. 

To use regular expressions with GlideQuery, you can utilize the `addQuery` method with the `REGEX` operator. Here's a simple example:

```javascript
var gr = new GlideRecord('incident');
gr.addQuery('short_description', 'REGEX', '[0-9]{3}');
gr.query();
while (gr.next()) {
    // your code here
}
```

In this example, the GlideQuery is looking for all incidents where the 'short_description' field contains a sequence of three digits (represented by the regular expression `[0-9]{3}`). The `{3}` quantifier means exactly three occurrences of the preceding expression, which in this case is `[0-9]` representing any digit from 0 to 9.

Another example would be searching for any records that do not match a specific pattern. You can do this by using `NOT REGEX`:

```javascript
var gr = new GlideRecord('incident');
gr.addQuery('short_description', 'NOT REGEX', '[0-9]{3}');
gr.query();
while (gr.next()) {
    // your code here
}
```

In this example, the GlideQuery is looking for all incidents where the 'short_description' field does not contain a sequence of three digits.

Please note that using `REGEX` in a query can be resource-intensive and may degrade system performance, so use it wisely and only when necessary. 

**Question**: What happens when you pass nothing to toArray()? 

**Answer**: The `toArray()` method in GlideQuery without passing any arguments will return all the records from the table in the form of an array of objects. Each object in the array represents a single record from the table, and the properties of the object are the fields of the record.

Here is an example of how you use the `toArray()` method:

```javascript
var users = new GlideQuery('sys_user')
    .select('user_name', 'first_name', 'last_name')
    .where('active', true)
    .toArray();

// users is now an array of objects, each representing an active user.
// Each object has properties user_name, first_name, and last_name.
```

In this example, the `toArray()` method returns an array of objects, each representing an active user. Each object has properties `user_name`, `first_name`, and `last_name` which are the fields selected by the `select()` method.

Please note that if no specific fields are selected using the `select()` method, by default, `toArray()` method will return all fields of the record. 

Also, remember that GlideQuery returns data asynchronously, so you need to handle the Promise returned by `toArray()` appropriately in your code. 

