---
layout: page
title: Elastic Git
date: 2014-11-19
---

[Elastic Git][eg] is a library we've developed to provide a single interface
when working with data that's distributed with [Git][git] and indexed
in [Elasticsearch][es]. It is built around Mozilla's [elasticutils][eu]
library.

Elastic Git (EG from now on) is built around the concept of
[declarative models][model].

Here is what a model looks like.

{% gist smn/306ddb1c3a4654ec4a7f model.py %}

We declare a model as a thing we want to work with and then define attributes
that that model has. In this case it is a name as a bit of text and an age
as an integer.

To code for all this is available [online](https://gist.github.com/smn/306ddb1c3a4654ec4a7f),
the samples in this document all come from there.

First to get working with [EG][eg] we need to get an environment up
and running, the following code will set that up for you:

{% gist smn/306ddb1c3a4654ec4a7f setup.sh %}

Note that you also need to have [Elasticsearch][es] running.

# The workspace

The workspace is an object that EG provides which provides a single point
of access to your data in [Git][git] and [Elasticsearch][es].

API documentation for it is [available online][eg] but here is a simple
example of creating a repository called *the_repository* and saving an
instance of a model in it:

{% gist smn/306ddb1c3a4654ec4a7f workspace1.py %}

What happens here is that under the hood we are generating a JSON file that
represents the model and the data, here is what it looks like in the
repository we've created:

{% gist smn/306ddb1c3a4654ec4a7f workspace1.log %}

The [UUID][uuid] is automatically generated and is the identifier with which
we refer to individual bits of data, both in applications and in git and
elasticsearch.

You will notice that it also has some extra metadata under the *_version* key
which provides some hints about what package & version of that package
generated the JSON data.

# Querying the workspace

Now that we have the ability to save things to the workspace, we can query
it as well. We've built a wrapper around [elasticutils][eu] that provides
the same interface and query API.

{% gist smn/306ddb1c3a4654ec4a7f workspace2.py %}

Running that will print the following:

{% gist smn/306ddb1c3a4654ec4a7f workspace2.log %}

Similarly we can generate & save more people entries:

{% gist smn/306ddb1c3a4654ec4a7f workspace3.py %}

And then query them:

{% gist smn/306ddb1c3a4654ec4a7f workspace4.py %}

Which would print out the following:

{% gist smn/306ddb1c3a4654ec4a7f workspace4.log %}

[Elasticutils][eu] queries return [MappingTypes][mapping_type] which are
objects reflecting the data that was indexed in Elasticsearch. EG provides
a mapping between objects in Elasticsearch and in Git and as a result you can
ask a result from Elasticsearch to return the object from Git.

{% gist smn/306ddb1c3a4654ec4a7f workspace5.py %}

When running this you will see the following:

{% gist smn/306ddb1c3a4654ec4a7f workspace5.log %}

What is returned by the *get_object()* function is the instance of the
*Person* class that was saved with *workspace.save()*.

# More complex querying

[Elasticutils][eu] allows for fairly complex query construction, and, if all
else fails, allows you to provide raw queries as input.

Here is an example of an *OR* query using the *F()* filter helper:

{% gist smn/306ddb1c3a4654ec4a7f workspace6.py %}

When run that prints out people with age 10 or name Simon:

{% gist smn/306ddb1c3a4654ec4a7f workspace6.log %}

At this point I would recommend the following reading:

- [Searching in Elasticutils](https://elasticutils.readthedocs.org/en/latest/searching.html#searching)
- [Elastic Git Workspace API](http://elastic-git.readthedocs.org/en/latest/managers.html#elasticgit.manager.Workspace)

[eg]: http://elastic-git.rtfd.org
[git]: http://git-scm.com
[es]: http://elasticsearch.org
[eu]: http://elasticutils.rtfd.org
[model]: http://en.wikipedia.org/wiki/Conceptual_model_(computer_science)
[uuid]: http://en.wikipedia.org/wiki/Universally_unique_identifier
[mapping_type]: https://elasticutils.readthedocs.org/en/latest/mappingtype.html