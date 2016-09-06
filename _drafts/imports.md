---
layout: post
title:  "What is __init__.py? and what should I put in it?"
date:   2016-05-09 08:21:54 -0700
---

# What is \_\_init\_\_.py?

A google search leads us to [stackoverflow][1] and then the [python documentation][2].  The gist is that `__init__.py` is used to indicate that a directory is a python package. (A quick note about packages vs. modules from the [python docs][3]: "From a file system perspective, packages are directories and modules are files.").  Okay, so if we want a folder to be considered a python package, we need to include a `__init__.py` file.  But what should we put init?

# What to put in it

(insert pic?)

There is a range of options of what to put in an `__init__.py` file.  On the minimalist side of the spectrum, an `__init__.py` is allowed to be totally empty.  Moving slightly away from this, while still keeping things simple, you can use an `__init__.py` only for determining the import order.  Another step up on the spectrum, you can use the `__init__.py` file to define the API of the package by defining the functions this package exposes.  And the final step is you can actually just define your entire package in the `__init__.py`.

Which of those options is right for a particular project depends. I'll dig into the pro's an cons of each of these 4 approaches and give examples of them in the wild in the rest of the post.

# #1 Leave it Empty

This approach is the simplest to communicate and the simplest to enforce.  There is no gray area about not including anything in an `__init__.py`.  The file will serve it's purpose of indicating the folder should be considered a python package, and nothing else.

One disadvantage of this approach is that it fails to take advantage of the special status of the `__init__.py` file.  The code in the file will be executed in the course of importing any of the packages submodules.  For instance, if we have a project with the following directory structure:
```
└── foo
    ├── __init__.py
    ├── a.py
    ├── b.py
    └── bar
        └── __init__.py
```

And we want to import the "a" module, the statement `from foo import a` looks in the `foo` directory, sees the `__init__.py`.  It knows to treat `foo` as a package, and it executes it's `__init__.py`, then looks for how to import `a`.  (You can verify this behavior by recreating this directory structure and putting print statements in the files.  If you do `from foo import c`, you'll get an `ImportError`, but not after the print statement in `foo/__init__.py` executes.  If you are interested in digging into the python source code, the code for `importlib` is available on [github][5].  Also the spec for the generic Importer Protocol is in [PEP-302][6]).  By leaving our `__init__.py` file blank, we miss out on the opportunity to leverage this.

Another disadvantage is related to namespaces.  In the directory structure listed above, importing `foo` anywhere will be useless. In order to access any of our actual code, we have to import sub modules.  In addition to making import statements longer, naming things is hard.  It is unfortunate to come up with a great name for a package or a sub-package and then also need to come up with good names for sub-modules since that is what you will end up referring to.

An example in the python source of this approach being used is in [urllib][7].


# #2 Enforce Import Order
This is what mssaxm over at axialcorps.com recommends in a post titled [5 Simple Rules For Building Great Python Packages][4].  This approach takes advantage of the special behavior of `__init__.py` while still keeping the file simple.  This approach really shines if your sub-modules have some static initialization.  For example, let's say `a.py` writes a config file when it is imported, and `b.py` reads from that file.  We could have our `__init__.py` ensure that `a.py` is always run before `b.py` by having it's contents be:
```
import a
import b
```
Then when we run `import foo.b`, it is guaranteed that `a.py` would be executed. (This dependency example is a bit contrived; I do not mean to suggest that sub-modules should make a habit of writing out files on import.)

Since this approach does not allow non-import code in the `__init__.py`, it seems to suffer from the namespace issue described in #1 above.  However, this can be circumvented by importing member from individual packages.  For instance, if we had a `my_func` that we wanted to be able to access as `import foo; foo.my_func()`, we could put `my_func` in `a.py` and then have our `__init__.py` be `from a import my_func`.

An example of this approach being used is the [fsq][8] package described by in [the post][4] I mentioned above.


# #3 Define API

In this approach, the `__init__.py` file houses the most visible functionality for the package.  It pieces together the functionality from the sub-modules.  This approach has the advantage of providing a good starting point to look into a package, and makes it clear what the top level functionality is.

The disadvantage is that your `__init__.py` file is more complicated.  The more complicated it gets, and the more deeply nested your package structure gets, the greater the risk of this causing problems.  Remember that importing a deeply nested package executes the `__init__.py` of every parent package.

Another disadvantage of this approach is that it can be difficult to decide what deserves to be in the `__init__.py` vs. in a sub-module.  As the file gets bigger and more complex, a call will need to be made about when to pull things out.

An example of this approach in python library code is in the [json][9] module.  The `__init__.py` file exposes the `dump`, `dumps` and `loads` functions which rely on functionality defined in sub-modules.


# #4 The Whole Package

The final approach is to put the entire package in the `__init__.py` file.  This can work well for small packages. It avoids needing to come up with a bunch of new names.

As the package gets larger however, a single file package can become unwieldy.

An example of this approach is [collections][10] module.  (Although, technically it does have one sub-module.)


# Conclusion

So what should you put in your `__init__.py`?  It depends on the project.  Hopefully the information in this post can help you assess the pro's and con's of each of these approaches.  For a guide on other general things to think about, I found a guide called [Structuring Your Project][11] on python-guide.org to be very helpful.

[1]:http://stackoverflow.com/questions/448271/what-is-init-py-for
[2]:https://docs.python.org/3/tutorial/modules.html#packages
[3]:https://docs.python.org/2.7/reference/simple_stmts.html#import
[4]:https://axialcorps.com/2013/08/29/5-simple-rules-for-building-great-python-packages/
[5]:https://github.com/python/cpython/blob/master/Lib/importlib/_bootstrap.py#L926
[6]:https://www.python.org/dev/peps/pep-0302/
[7]:https://github.com/python/cpython/tree/master/Lib/urllib
[8]:https://github.com/axialmarket/fsq/blob/master/fsq/__init__.py
[9]:https://github.com/python/cpython/blob/master/Lib/json/__init__.py
[10]:https://github.com/python/cpython/blob/master/Lib/collections/__init__.py
[11]:http://docs.python-guide.org/en/latest/writing/structure/
