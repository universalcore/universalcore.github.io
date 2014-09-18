---
layout: page
title: Post Management and Publishing Infrastructure
date: 2014-9-18
---

Universal Core's guiding principles around decentralization have
significant implications on how content is authored, distributed and
published.

Feature list for this work is maintained using [GitHub milestones](https://github.com/praekelt/unicore/milestones/15%20-%2026%20September).

***

Publishing Infrastructure
-------------------------

The implications we are currently aware of are:

* Content is managed in a graph like network of peer nodes.
* Each node exposes an API for content it is aware of.
* Each node exposes a UI for managing the content it is aware of.
* The network of nodes is nonhierarchical.
* Nodes can notify each other of content being published.
* Nodes can decide to pull in content published by peer nodes.
* Nodes are autonomous.

This has lead to the current design, which is still highly experimental:

1. All content is managed and stored in a [Git](http://git-scm.com) repository.
2. Every bit of content is a [JSON](http://json.org) object identified by a
   [UUID](http://en.wikipedia.org/wiki/Universally_unique_identifier).
3. Every change to content results in a commit in the Git content repository.
4. Changes can be pushed to a repository hosted elsewhere on a regular basis.
5. When being deployed, nodes can be configured to track changes to
   Git content repositories.
6. Nodes can be notified of changes to a repository, prompting them to
   pull in the changes, resulting in a content refresh on the node.
7. All content is queried via a dedicated search backend, currently defaulting
   to [Elastic Search](www.elasticsearch.org).
8. Nothing is ever deleted, Git provides a full history & audit log of
   changes made.

Post Management
---------------

We are working towards supporting the minimum set of features required
for launching a content site within Internet.org.

We have a very basic interface for managing posts, categories, language
and home page publishing. This interface writes the data to a Git
repository. The configured nodes are notified in an asynchronous manner
and each pulls in the newly published content for local publishing.
