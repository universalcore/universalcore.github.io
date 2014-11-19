---
layout: page
title: Elasticsearch & Querying
date: 2014-11-19
---

Universal Core doesn't have a database for content in the traditional sense,
we store our data in JSON files.

Using JSON for storage is great because the format is human readable and
fairly light weight but querying those files without any kind of specialised
index is impractical.

As a result we are using a combination of [Git][git] for distribution and
[Elasticsearch][Elasticsearch] for querying.

This works as follows during a deploy (of both the application and the content):

1. Pull the new data from the Git repository
2. Iterate over all of the changes (additions, removals, modifications) and
   store, update or remove these from an [Elasticsearch][Elasticsearch] index.
3. Query the data using [elastic-git][elastic-git]

## Working with Elasticsearch

While Elasticsearch is a search engine it doubles up as a really good
database for structured content. It understands JSON and provides good tools
for indexing and querying JSON content, a perfect fit for our needs.

Elasticsearch expects data to have a "type" assigned and stores this
information in a index.

If you're coming from a background with relational databases like MySQL or
Postgresql then think of the "index" as your database and the "type" as your
table.

Creating an index:

    $ curl -XPOST http://localhost:9200/contacts
    {"acknowledged":true}

Adding items to the index

    $ curl -XPUT http://localhost:9200/contacts/friends/foo -d '{
        "_type": "friend",
        "name": "Foo",
        "age": 25
    }'
    {"_index":"contacts","_type":"friends","_id":"foo","_version":1,"created":true}


    $ curl -XPUT http://localhost:9200/contacts/friends/bar -d '{
            "_type": "friend",
            "name": "Bar",
            "age": 25
    }'
    {"_index":"contacts","_type":"friends","_id":"bar","_version":1,"created":true}

Now we can query them. Note that we're quoting the URL now because ampsersands
have special meanings in Bash:

    $ curl -XGET "http://localhost:9200/contacts/_search?q=name:Bar&pretty=True"
    {
      "took" : 1,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
      },
      "hits" : {
        "total" : 1,
        "max_score" : 1.0,
        "hits" : [ {
          "_index" : "contacts",
          "_type" : "friends",
          "_id" : "bar",
          "_score" : 1.0,
          "_source":{
                "_type": "friend",
                "name": "Bar",
                "age": 25
        }
        } ]
      }
    }

The elastic search [query options][query options] are quite powerful, here is an example of a
range ``age:[20 TO 30]`` query (it needs to be urlencoded):

    $ curl -XGET "http://localhost:9200/contacts/_search?q=age:%5B20%20TO%2030%5D&pretty=True"
    {
      "took" : 2,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
      },
      "hits" : {
        "total" : 2,
        "max_score" : 1.0,
        "hits" : [ {
          "_index" : "contacts",
          "_type" : "friends",
          "_id" : "foo",
          "_score" : 1.0,
          "_source":{
            "_type": "friend",
            "name": "Foo",
            "age": 25
        }
        }, {
          "_index" : "contacts",
          "_type" : "friends",
          "_id" : "bar",
          "_score" : 1.0,
          "_source":{
                "_type": "friend",
                "name": "Bar",
                "age": 25
        }
        } ]
      }
    }

## Querying this from Python

Mozilla has written a great library for interacting with data in Elasticsearch.
This library is called [elasticutils][elasticutils].

Create a virtualenv somewhere and let's access the data we've just put in
Elasticsearch from Python:

    $ virtualenv ve
    $ source ve/bin/activate
    (ve)$ pip install elasticutils
    ... snip ...
    Successfully installed elasticutils elasticsearch six urllib3
    Cleaning up...

Now start Python in your virtualenv:

    (ve)$ python
    Python 2.7.6 (default, Dec 22 2013, 09:30:03)
    [GCC 4.2.1 Compatible Apple LLVM 5.0 (clang-500.2.79)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.

Start querying the data we put in Elasticsearch with curl earlier:

    >>> from elasticutils import S
    >>> S().indexes('contacts').count()
    2
    >>> S().indexes('contacts').filter(name='bar').count()
    1
    >>> S().indexes('contacts').filter(name='foo').count()
    1

Note that we only get results back if we lowercase "Bar" to "bar" and
"Foo" to "foo". This has got to do with Elasticsearch's default [analyzers](analyzers).

Read up on those because those are important when we need to query for
case sensitive fields.

The `type` roughly reflects what in a normal database you would call a
table. As you can see Elasticsearch knows we have 2 friends and zero haters.

    >>> S().indexes('contacts').doctypes('friends').count()
    2
    >>> S().indexes('contacts').doctypes('haters').count()
    0

We can do range queries as well:

    >>> S().indexes('contacts').doctypes('friends').filter(age__gte=20, age__lt=30).count()
    2

And fetch the results:

    >>> for result in S().indexes('contacts').doctypes('friends').filter(age__gte=20, age__lt=30):
    ...     print result.name, result.age
    ...
    Foo 25
    Bar 25

And specify a custom ordering

    >>> for result in S().indexes('contacts').doctypes('friends').filter(age__gte=20, age__lt=30).order_by('name'):
    ...     print result.name, result.age
    ...
    Bar 25
    Foo 25

This just scratches the surface on what [Elasticsearch][Elasticsearch] and
[elasticutils][elasticutils] can do. Please check the invididual sites for
more documentation on analyzing, indexing and querying.

[git]: http://git-scm.com
[elastic-git]: http://elastic-git.rtfd.org
[Elasticsearch]: http://www.elasticsearch.org
[query options]: http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html
[elasticutils]: http://elasticutils.readthedocs.org
[analyzers]: http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/analysis-analyzers.html
