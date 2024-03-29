---
Title: RediSearch quick start
linkTitle: Quick start
description: RediSearch quick start on how to index hash documents and run search queries.
weight: 20
alwaysopen: false
categories: ["Modules"]
module: RediSearch
aliases: /rs/getting-started/creating-database/redisearch/
---

This quick start shows you how to index documents stored as Redis [hashes](https://redis.io/docs/manual/data-types/#hashes) and run search queries against the index. To learn how to index and search JSON documents, see the [Search JSON quick start]({{<relref "/modules/redisearch/search-json-quickstart">}}).

## Prerequisites

For this tutorial, you need:

- A Redis database with the RediSearch module enabled. You can use either:

    - A [Redis Cloud]({{<relref "/modules/modules-quickstart.md">}}) database

    - A [Redis Enterprise Software]({{<relref "/modules/install/add-module-to-database">}}) database

- [`redis-cli`]({{<relref "/rs/references/cli-utilities/redis-cli">}}) command-line tool

- [`redis-py`](https://github.com/redis/redis-py) client library v4.0.0 or later

## RediSearch with `redis-cli`

To begin, [connect to your database]({{<relref "/rs/references/cli-utilities/redis-cli#connect-to-a-database">}}) with `redis-cli`.

### Create an index

The [`FT.CREATE`](https://redis.io/commands/ft.create) command lets you create an index. It indexes Redis [hashes](https://redis.io/docs/manual/data-types/#hashes) by default. However, as of RediSearch v2.2, you can also [index JSON documents]({{<relref "/modules/redisearch/search-json-quickstart">}}) if you have the [RedisJSON]({{<relref "/modules/redisjson">}}) module enabled.

To set the language used for indexing and [stemming](https://redis.io/docs/stack/search/reference/stemming), include the optional `LANGUAGE` keyword. If you do not specify a language, it defaults to English. See [Supported languages](https://redis.io/docs/stack/search/reference/stemming/#supported-languages) for additional language options.

When you define an index, you also need to define the schema, or structure, of the data you want to add to the index.

The schema for this example has four fields: 
- `title` (`TEXT`)
- `body` (`TEXT`)
- `url` (`TEXT`)
- `visits` (`NUMERIC`)

To define the schema and create the index, run:

```sh
127.0.0.1:12543> FT.CREATE database_idx PREFIX 1 "doc:" LANGUAGE english SCORE 0.5 SCORE_FIELD "doc_score" SCHEMA title TEXT body TEXT url TEXT visits NUMERIC
```

This command indexes all hashes with the prefix "`doc:`". It will also index any future hashes that have this prefix.

By default, all documents have a score of 1.0. However, you can configure an index's default document score with the <nobr>`SCORE <default_score>`</nobr> option.

If you want to allow document-specific scores, use the <nobr>`SCORE_FIELD <score_attribute>`</nobr> option.

### Add documents

After you create an index, you can add documents to the index.

Use the [`HSET`](https://redis.io/commands/hset) command to add a [hash](https://redis.io/docs/manual/data-types/#hashes) with the key "`doc:1`" and the fields:

- title: "RediSearch"
- body: "RediSearch is a powerful indexing, querying, and full-text search engine for Redis"
- url: <https://redis.com/modules/redis-search/>
- visits: 108

```sh
127.0.0.1:12543> HSET doc:1 title "RediSearch" body "RediSearch is a powerful indexing, querying, and full-text search engine for Redis" url "<https://redis.com/modules/redis-search/>" visits 108
(integer) 4
```

If you want to make a document appear higher or lower in the search results, add a document-specific score.

To do so, use the `score_attribute` set by the `SCORE_FIELD` option during index creation (`score_attribute` is `doc_score` in this case):

```sh
127.0.0.1:12543> HSET doc:2 title "Redis" body "Modules" url "<https://redis.com/modules>" visits 102 doc_score 0.8
(integer) 5
```

### Search an index

Use [`FT.SEARCH`](https://redis.io/commands/ft.search) to search the index for any documents that contain the words "search engine":

```sh
127.0.0.1:12543> FT.SEARCH database_idx "search engine" LIMIT 0 10
1) (integer) 1
2) "doc:1"
3) 1) "title"
   2) "RediSearch"
   3) "body"
   4) "RediSearch is a powerful indexing, querying, and full-text search engine for Redis"
   5) "url"
   6) "<https://redis.com/modules/redis-search/>"
   7) "visits"
   8) "108"
```

### Drop an index

To remove the index without deleting any associated documents, run the [`FT.DROPINDEX`](https://redis.io/commands/ft.dropindex) command:

```sh
127.0.0.1:12543> FT.DROPINDEX database_idx
OK
```

### Auto-complete

You can use RediSearch suggestion commands to implement [auto-complete](https://redis.io/docs/stack/search/design/overview/#auto-completion).

{{<note>}}
Active-Active databases do not support RediSearch suggestions.
{{</note>}}

Use [`FT.SUGADD`](https://redis.io/commands/ft.sugadd) to add a phrase for the search engine to suggest during auto-completion:

```sh
127.0.0.1:12543> FT.SUGADD autocomplete "primary and caching" 100
"(integer)" 1
```

Test auto-complete suggestions with [`FT.SUGGET`](https://redis.io/commands/ft.sugget):

```sh
127.0.0.1:12543> FT.SUGGET autocomplete "pri"
1) "primary and caching"
```

## RediSearch with Python

If you want to use RediSearch within an application, these [client libraries](https://oss.redis.com/redisearch/Clients/) are available.

The following example uses the Redis Python client library [redis-py](https://github.com/redis/redis-py), which supports RediSearch commands as of v4.0.0.

This Python code creates an index, adds documents to the index, displays search results, and then deletes the index:

```python
import redis
from redis.commands.search.field import TextField, NumericField
from redis.commands.search.indexDefinition import IndexDefinition
from redis.commands.search.query import Query

# Connect to a database
r = redis.Redis(host="<endpoint>", port="<port>", 
                password="<password>")

# Options for index creation
index_def = IndexDefinition(
                prefix = ["py_doc:"],
                score = 0.5,
                score_field = "doc_score"
)

# Schema definition
schema = ( TextField("title"),
           TextField("body"),
           TextField("url"),
           NumericField("visits")
)

# Create an index and pass in the schema
r.ft("py_idx").create_index(schema, definition = index_def)

# A dictionary that represents a document
doc1 = { "title": "RediSearch",
         "body": "RediSearch is a powerful indexing, querying, and full-text search engine for Redis",
         "url": "<https://redis.com/modules/redis-search/>",
         "visits": 108
}

doc2 = { "title": "Redis",
         "body": "Modules",
         "url": "<https://redis.com/modules>",
         "visits": 102,
         "doc_score": 0.8
}

# Add documents to the database and index them
r.hset("py_doc:1", mapping = doc1)
r.hset("py_doc:2", mapping = doc2)

# Search the index for a string; paging limits the search results to 10
result = r.ft("py_idx").search(Query("search engine").paging(0, 10))

# The result has the total number of search results and a list of documents
print(result.total)
print(result.docs)

# Delete the index; set delete_documents to True to delete indexed documents as well
r.ft("py_idx").dropindex(delete_documents=False)
```

Example output:
```sh
$ python3 quick_start.py 
1
[Document {'id': 'py_doc:1', 'payload': None, 'visits': '108', 'title': 'RediSearch', 'body': 'RediSearch is a powerful indexing, querying, and full-text search engine for Redis', 'url': '<https://redis.com/modules/redis-search/>'}]
```

## More info

- [RediSearch commands]({{<relref "/modules/redisearch/commands">}})
- [RediSearch query syntax](https://redis.io/docs/stack/search/reference/query_syntax/)
- [Details about RediSearch query features](https://redis.io/docs/stack/search/reference/)
- [RediSearch client libraries](https://redis.io/docs/stack/search/clients/)
- [Search JSON quick start]({{<relref "/modules/redisearch/search-json-quickstart">}})
