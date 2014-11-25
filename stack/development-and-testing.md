---
layout: page
title: Development and Testing
date: 2014-11-25
---

Writing new features for Unicore happens in lock step with writing tests
for these new features. The main reason behind this is to manage the
complexity of software. As software grows, often organically, it becomes
increasingly difficult to think through all of possible implications of
a single change through the entire system.

Writing tests gives us smaller chunks to reason about, sizes we are
generally more easily able to keep in our heads.

We won't explain how to write tests ([here is a fairly good tutorial][tutorial]
on this) but we'll give a run down of what tools and utilities
are available when writing tests for Unicore.

All tests in Universal Core subclass the [UnicoreTestCase][UnicoreTestCase]
which provides 3 helper functions:

* `mk_workspace` for creating adhoc [EG][eg] workspaces.
* `create_categories` for creating `Category` objects in an [EG][eg] workspace.
* `create_pages` for creating `Page` objects in an [EG][eg] workspace.

# Writing a test

Here's a very contrived example of a test, it simple creates a workspace
and then tests that one is actually created:

{% gist smn/5e78b96cbc8d46c6a261 test1.py %}

The output of that is

{% gist smn/5e78b96cbc8d46c6a261 test1.log %}

Now the value of this test is very limited, we can do better.

Let's create a file called `models.py` that defines an [EG][eg] model
for us to work with in the workspace:

{% gist smn/5e78b96cbc8d46c6a261 models.py %}

In our test we can import that using `from models import Person`:

{% gist smn/5e78b96cbc8d46c6a261 test2.py %}

Running this you'll notice the following test failure:

{% gist smn/5e78b96cbc8d46c6a261 test2.log %}

This is because [Elasticsearch's Indexes][indexes] are refreshed
asynchronously at an interval. Generally what happens in tests is that your
index is not yet refreshed by the time you query it. Manually refreshing it
resolves that and makes this test pass:

{% gist smn/5e78b96cbc8d46c6a261 test3.py %}

Passing test!

{% gist smn/5e78b96cbc8d46c6a261 test3.log %}

# Using a test runner

There are a lot of scenarios where running the tests straight in Python
using `unittest.main()` is the easiest way to quickly write a test.

However, when writing a body of software that is larger than a single file
it is usually easier to use a test runner. For our code bases we prefer
[py.test][pytest].

Install it in your virtualenv with pip:

    (ve)$ pip install pytest

Now run the following command:

    (ve)$ py.test test3.py --verbose

That will generate the following output:

    ======================================== test session starts =======
    platform darwin -- Python 2.7.6 -- py-1.4.26 -- pytest-2.6.4 -- ...
    collected 2 items

    test3.py::ExampleTest::test_creating_a_person PASSED
    test3.py::ExampleTest::test_making_a_workspace PASSED

    ===================================== 2 passed in 0.56 seconds =====

[Py.test][pytest] gives you many command line options that allow you to
select a full suite of tests or a single test.

`-k` is the command line option for only running tests matching a certain
string:

    (ve)$ py.test test3.py -k creating_a_person --verbose
    ======================================== test session starts =======
    platform darwin -- Python 2.7.6 -- py-1.4.26 -- pytest-2.6.4 -- ...
    collected 2 items

    test3.py::ExampleTest::test_creating_a_person PASSED

    ================== 1 tests deselected by '-kcreating_a_person' =====
    ================== 1 passed, 1 deselected in 0.46 seconds ==========


# Debugging a test

Sometimes you'll find that you're simply failing to get a test passing.
At that point being able to look under the hood in the Git repository
to see what data's been created and indexed in Elasticsearch can be
incredibly helpful.

The [UnicoreTestCase][UnicoreTestCase] by default automatically destroys
and workspace after the test is in finished. However you can opt to
leave it lying around for inspection by setting the `KEEP_REPO` environment
variable:

    (ve)$ KEEP_REPO=1 py.test test3.py -k creating_a_person --verbose
    ======================================== test session starts =======
    platform darwin -- Python 2.7.6 -- py-1.4.26 -- pytest-2.6.4 -- ...
    collected 2 items

    test3.py::ExampleTest::test_creating_a_person PASSED

    ================== 1 tests deselected by '-kcreating_a_person' =====
    ================== 1 passed, 1 deselected in 0.46 seconds ==========

By default the [UnicoreTestCase][UnicoreTestCase] uses the `.test_repos`
directory as the working directory when creating workspaces.
With `KEEP_REPO` set it will leave the git repository there instead of
clearing it:

    (ve)$ ls .test_repos
    test_creating_a_person

We can look at the `Person` JSON files created:

    (ve)$ ls .test_repos/test_creating_a_person/models/Person/
    590849dd7ddb4ef38ff58afff893a6bd.json
    (ve)$ cat .test_repos/test_creating_a_person/models/Person/590849dd7ddb4ef38ff58afff893a6bd.json
    {
      "age": 25,
      "_version": {
        "package_version": "0.3.0",
        "language_version": "2.7.6",
        "language_version_string": "2.7.6 (default, Dec 22 2013, 09:30:03) \n[GCC 4.2.1 Compatible Apple LLVM 5.0 (clang-500.2.79)]",
        "language": "python",
        "package": "elastic-git"
      },
      "name": "foo",
      "uuid": "590849dd7ddb4ef38ff58afff893a6bd"
    }

Using the [EG][eg] `shell` we can now inspect the contents of this repository
from Python.

    (ve)$ PYTHONPATH=. eg-tools shell .test_repos/test_creating_a_person
    models does not look like a models module.
    Python 2.7.6 (default, Dec 22 2013, 09:30:03)
    [GCC 4.2.1 Compatible Apple LLVM 5.0 (clang-500.2.79)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    (InteractiveConsole)
    >>> from models import Person
    >>> workspace.S(Person).count()
    1
    >>> workspace.S(Person).filter(age=25).count()
    1
    >>> [person] = workspace.S(Person).filter(age=25)
    >>> print person.name
    foo
    >>> workspace.destroy()


[tutorial]: http://pymotw.com/2/unittest/index.html
[UnicoreTestCase]: https://github.com/universalcore/unicore-cms/blob/develop/cms/tests/base.py
[eg]: http://elastic-git.rtfd.org
[indexes]: http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/indices-refresh.html#indices-refresh
[pytest]: http://pytest.org
