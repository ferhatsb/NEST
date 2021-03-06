---
layout: default
title: Connecting
menu_section: core
menu_item: index
---


# Indexing

Indexing is as simple as:

	var post = new Post() { Id = 12, ... }
	client.Index<Post>(post);

of course C# is smart enough to infer Post so

	client.Index(post);

is sufficient. this will index post too `/[default index]/posts/12`. The typename`posts` is automatically inferred from the type.

if you need more control there are plenty of overloads, i.e:

	client.Index(post, "index", "type", "id");

## Asynchronous

Indexing asynchronously is as easy as:

	client.IndexAsync(post, (c) => /* called later */);


## Aditional parameters

You can pass aditional data using `IndexParameters`

	client.Index(post, new IndexParameters() { VersionType = VersionType.External, Version = "212" });

Similarly to force a wait for a refresh 

	client.Index(post, new IndexParameters() { Refresh = true });

## Bulk Indexing

Instead of passing `T` just pass `IEnumerable<T>` for both `Index()` or `IndexAsync()`. A zero copy approach that writes directly on the post stream is planned in a later version.

**Note**
For asynchronous commands there's a special connection setting which automatically semaphores threaded communication
to ES for you:

	var elasticSettings = new ConnectionSettings("127.0.0.1.", 9200)
							  .SetDefaultIndex("mpdreamz")
							  .SetMaximumAsyncConnections(20);

this ensures that at most there are 20 asynchronous connections to ES others are enqueued until a slot is 
available.

## Aditional parameters
Like the overloads just taking a `T` the `IEnumerable<T>` has alot of overloads taking in extra parameters. 

	client.Index(posts, new SimpleBulkParameters() { Refresh = true });

The reason the `IEnumerable<T>` overloads take a `SimpleBulkParameters` is because to pass item specific parameters you'll have to wrap `posts` in a `BulkParameters<T>` i.e:

	client.Index(posts.Select(p=>new BulkParameters<T>(p) { Version = p.Version }));

This will do a bulk index on posts but use each individual posts version. Again there's plenty of overloads to mix and match:

	var bulkParams = posts.Select(p=>new BulkParameters<T>(p) { Version = p.Version });
	client.Index(bulkParams , new SimpleBulkParameters() { Refresh = true });


 

