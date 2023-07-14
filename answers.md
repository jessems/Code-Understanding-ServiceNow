**Question**: How does GlideQuery work?

**Answer**: `GlideQuery` is a class in ServiceNow that provides a more modern and flexible way to perform database operations compared to the traditional `GlideRecord` and `GlideAggregate` APIs. It allows developers to build and execute database queries in a more intuitive and succinct manner, while also providing more advanced and powerful query features.

Here's a simple breakdown of how it works:

1. **Initialization**: You create a new `GlideQuery` object by passing the table name to its constructor:

    ```javascript
    var query = new GlideQuery('sys_user');
    ```

2. **Building the query**: You can then chain methods onto the `GlideQuery` object to build your query. Each method corresponds to a specific database operation or condition. For example, you can add filter conditions using the `where()` method:

    ```javascript
    query.where('active', true);
    ```

    You can also limit the number of records returned using the `limit()` method:

    ```javascript
    query.limit(10);
    ```

3. **Executing the query**: You can execute the query and retrieve the results using the `toArray()` or `toGlideRecord()` methods:

    ```javascript
    var results = query.toArray();
    ```

    ```javascript
    var glideRecord = query.toGlideRecord();
    glideRecord.query();
    while (glideRecord.next()) {
    	// Do something with the glideRecord
    }
    ```

Under the hood, the `GlideQuery` class maintains an internal representation of the query, which is updated each time you call a method to modify the query. This internal representation is used to generate the actual SQL query that is executed against the database when you call `toArray()` or `toGlideRecord()`.

In addition to the standard query operations provided by `GlideRecord`, `GlideQuery` also supports more advanced features like subqueries, complex join operations, and aggregation functions.

**Question**: How is GlideQuery implemented that it allows for you to chain methods on top of each other?

**Answer**: GlideQuery is designed to allow method chaining by returning a new GlideQuery object after each method call. This is made possible because each method call adds a new object to the 'plan' array representing the action to be taken, then returns a new GlideQuery with the updated 'plan'.

Let's take a look at the 'withAcls()' method as an example:

```javascript
GlideQuery.prototype.withAcls = function withAcls() {
	return new GlideQuery(
		this.table,
		this.plan.concat({
			type: 'withAcls'
		})
	);
};
```

In the code above, a new GlideQuery object is created and returned. The new GlideQuery has the same 'table' as the current GlideQuery, and a 'plan' array that includes all current plan steps plus a new step of the type 'withAcls'. This returned object can then be used for further method chaining.

So if you have a GlideQuery, you can chain methods like this:

```javascript
var users = new GlideQuery('sys_user')
	.withAcls()
	.limit(20)
	.orderByDesc('first_name')
	.select('first_name')
	.toArray(100);
```

Each method call in the chain adds a new step to the 'plan' and returns a new GlideQuery object, allowing for the next method in the chain to be called.

**Question**: What about the toArray() method, I don't understand why we need that. What does it do?

**Answer**: The `toArray()` method in GlideQuery is used to fetch the records from the ServiceNow database and returns an array of JavaScript objects representing the records. The objects in the array have properties and values corresponding to the fields and values of the records.

This method is terminal, meaning it executes the GlideQuery, fetches the records, and transforms them into an array of JavaScript objects. After calling `toArray()`, you cannot add more operations to the GlideQuery.

Here's a simple example:

```javascript
var users = new GlideQuery('sys_user')
	.select('name', 'email')
	.limit(10)
	.toArray();

users.forEach(function (user) {
	gs.info(user.name + ': ' + user.email);
});
```

In this example, `users` is an array of objects. Each object represents a user record and has `name` and `email` properties. The `forEach` loop is used to print the name and email of each user.

Behind the scenes, GlideQuery converts each GlideRecord it fetches into a JavaScript object. The returned JavaScript objects are plain data objects, they do not have the methods a GlideRecord has. If you need to call GlideRecord methods on the records, you should use `forEach()`, `map()`, or `reduce()` instead.

It's important to note that, by default, GlideQuery only fetches the values of the fields you specify with `select()`. If you call `toArray()` without calling `select()` first, the returned objects will only have `sys_id` field.

Also, remember to use `limit()` method when using `toArray()`, to avoid fetching a large number of records and potentially running out of memory.

**Question**: How can I use regex with GlideQuery?

**Answer**: GlideQuery does not natively support regular expressions. The usual way to search for a pattern within a string field in ServiceNow is to use the `LIKE` operator or the `MATCHES` operator within an encoded query string.

However, you can use the `addEncodedQuery` method of the GlideRecord object to add a regular expression to your GlideQuery. Here is an example:

```javascript
var gr = new GlideRecord('incident');
gr.addEncodedQuery('short_descriptionLIKE^abc'); // This will search for the string 'abc' within the short_description field
gr.query();
while (gr.next()) {
	// perform operations
}
```

In the above example, `LIKE` operator is used to perform a case-insensitive search for a string within a column.

Please note that using regular expressions in database queries can have performance implications, especially on large tables. Always ensure your regular expressions and queries are optimized and perform well.

If you need more advanced regular expression functionality, you might need to fetch the data first and then use JavaScript's built-in regular expression functionality to further process the data.
