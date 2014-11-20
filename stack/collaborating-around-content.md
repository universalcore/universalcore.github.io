---
layout: page
title: Collaborating around Content
date: 2014-11-20
---

There are by now a fair amount of Git repositories with JSON data. Most of
them are under the [Universal Core GitHub page][content-repos] but
there's not reason they all need to. They can be hosted anywhere.

We call these the content repositories. The authoring environment creates
the JSON files and pushes these to the repositories. The applications then
pull in these changes, index them in Elasticsearch and display that
content on the site.

This document explains how one would use the content in a content
repository.

# Version management

When working with relational databases there are constraints enforced on
what the data needs to look like, this is the schema. The schema determines
what columns are in which tables and what values should be stored in those
columns.

JSON is a very flexible format, it does not enforce any constraints on how
we store our data. This gives us a great amount of flexibility but also
potentially very many opportunities for us to get ourselves into trouble.
Every bit of data has the potential of being a unique and special
snowflake that needs specialized bits of code to read and write. When trying
to keep code maintainable this is not what we want.

We use [Apache Avro][avro] schema definitions to help us define the structure
of our JSON data. Every application that writes JSON data is expected to make
an [Avro][avro] schema file available. The schema defines the fields and
field types of the models being used.

Universal Core content repositories at the moment know of two types of models,
a Page and a Category.

**Category**

{% gist smn/5d71300ef8e2e016802a Category.avro.json %}

**Page**

{% gist smn/5d71300ef8e2e016802a Page.avro.json %}

This is important to make available because [EG][eg] can automatically
generate the model definitions based on these schema files.

[EG][eg] ships a small command line utility called *eg-tools* which
provides helpers for performing various tasks on JSON data, including
automatically generating the code for models based on a schema.

Here we automatically generate the *Category* model definition from the
*Category* Avro schema file.

{% gist smn/5d71300ef8e2e016802a generate-models1.log %}

While this works very well, you'll notice a few things:

1. The *uuid* field is a *TextField* while we need that to be a *UUIDField*
   as that adds specific behaviour around automatically generating UUIDs
   if it doesn't exist.

2. It defines a namespace, in Python we use this as this model's module.

3. The *uuid* field has a fallback type specified. This is [Avro's][avro]
   way of handling schema evolutions. If it loads a JSON dictionary that
   has no *uuid* field it will instead use the value from the *id* field.
   This allows us to make changes to fields as our software grows while
   still being able to read the data written by older versions.

Before we can start using this model we need to make sure we place the
generated model file into the correct module folder and ensure that we
are using *UUIDFields* instead of *TextFields* for the *uuid* field type.

Here is how we do that with *eg-tools*:

{% gist smn/5d71300ef8e2e016802a generate-models2.sh %}

This results in a *models.py* file in the *unicore/content* directory that
looks like this:

{% gist smn/5d71300ef8e2e016802a models.py %}

Now we have a model definition that we can use in [EG][eg] to query the data.

# Interacting with the data

What running the follow commands clones a sample content repository and
indexes the data in Elasticsearch:

{% gist smn/5d71300ef8e2e016802a clone-repo.sh %}

This should print something like the following:

{% gist smn/5d71300ef8e2e016802a clone-repo.log %}

Next we can fire up a shell with python shell with [EG] loaded to explore our
freshly indexed content! The [EG] *shell* helper automatically detects
what data is in your repository, creates a workspace for you and loads
the relevant Python model definitions we generated earlier from the
[Avro][avro] spec:

{% gist smn/5d71300ef8e2e016802a shell.log %}

Here are some example queries:

{% gist smn/5d71300ef8e2e016802a filter_categories.py %}

Now we have list of UUIDs for categories and we can ask [EG] for pages
that have that category set as its *primary_category*:

{% gist smn/5d71300ef8e2e016802a filter_pages.py %}

At this point I would recommend starting to read the source code of the
[unicore-cms][unicore-cms] repository as you now have all the information
you need in order to understand how it accesses the data stored in Git.

[content-repos]: https://github.com/universalcore/?query=unicore-cms-content
[avro]: http://avro.apache.org/docs/1.7.7/spec.html
[eg]: http://elastic-git.rtfd.org
[unicore-cms]: http://github.com/universalcore/unicore-cms