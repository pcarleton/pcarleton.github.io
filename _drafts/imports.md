---
layout: post
title:  "What's in __init__.py?"
date:   2016-05-09 08:21:54 -0700
---

First - what is __init__.py? What does it do? In what contexts do you need it?
When is it executed? (Whenever you import a module, whenever you import a sub package)

How can it be used?
global initialization
an entire package
to expose the package to users
empty

what is __all__?
what gets imported when you say "from X import \*"

How do people use it in practice?

one example of a python package

urllib:
https://github.com/python/cpython/tree/master/Lib/urllib
empty __init__.py

json:
https://github.com/python/cpython/blob/master/Lib/json/__init__.py
dump, dumps, load, loads
default encoder global object
relative imports of encoder and decoder

collections:
weird leading underscore imports...
https://github.com/python/cpython/blob/master/Lib/collections/__init__.py

one example of an open source package
(requests, urllib3, pandas)

requests:
https://github.com/kennethreitz/requests/blob/master/requests/__init__.py
sets up logger
imports many relative packages.

urllib3:
https://github.com/shazow/urllib3/blob/master/urllib3/__init__.py
imports, logger, author, license, warnings

https://github.com/shazow/urllib3/blob/master/urllib3/util/__init__.py
relative imports __all__

pandas:
https://github.com/pydata/pandas/blob/master/pandas/__init__.py

reads like an import script, defining orders of imports.

What's right for my project?
Be Consistent
Small project - cram everything in __init__.py


Talk about setup.py? pip tools and easy install?  Probably should avoid getting into that.
Django monolith?  No stay focused.



Python imports
__init__.py

https://axialcorps.com/2013/08/29/5-simple-rules-for-building-great-python-packages/

http://choosealicense.com/

http://docs.python-guide.org/en/latest/writing/structure/
ravioli code

Any directory with an __init__.py file is considered a Python package. The different modules in the package are imported in a similar manner as plain modules, but with a special behavior for the __init__.py file, which is used to gather all package-wide definitions

When the project complexity grows, there may be sub-packages and sub-sub-packages in a deep directory structure. In this case, importing a single item from a sub-sub-package will require executing all __init__.py files met while traversing the tree.

http://docs.python-guide.org/en/latest/writing/structure/#context-managers
Cool, now I understand the "with" keyword a little better.
Interestingly, they recommend a generator with try/finally.  I wonder if that leads
to the garbage scenario


https://www.python.org/dev/peps/pep-3101/
Discouragment of % operator


https://docs.python.org/3/library/functions.html#__import__
python 3 import semantics.  Same as 2.7? Yes.

https://docs.python.org/2.7/reference/simple_stmts.html#import
quote: A package can contain other packages and modules while modules cannot contain other modules or packages. From a file system perspective, packages are directories and modules are files.

https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_module
Seems like exec_module probably gets called for parent packages.


https://www.python.org/dev/peps/pep-0302/
quote: Deeper down in the mechanism, a dotted name import is split up by its components. For "import spam.ham", first an "import spam" is done, and only when that succeeds is "ham" imported as a submodule of "spam". >> The Importer Protocol operates at this level of individual imports. By the time an importer gets a request for "spam.ham", module "spam" has already been imported.


The __package__ attribute [8] must be set.

If the module is a Python module (as opposed to a built-in module or a dynamically loaded extension), it should execute the module's code in the module's global name space ( module.__dict__ ).

Here is a minimal pattern for a load_module() method:

# Consider using importlib.util.module_for_loader() to handle
# most of these details for you.
def load_module(self, fullname):
    code = self.get_code(fullname)
    ispkg = self.is_package(fullname)
    mod = sys.modules.setdefault(fullname, imp.new_module(fullname))
    mod.__file__ = "<%s>" % self.__class__.__name__
    mod.__loader__ = self
    if ispkg:
        mod.__path__ = []
        mod.__package__ = fullname
    else:
        mod.__package__ = fullname.rpartition('.')[0]
    exec(code, mod.__dict__)
    return mod

https://github.com/python/cpython/blob/master/Lib/importlib/_bootstrap.py#L926
These lines show the sanity check to ensure that parent is imported first.

https://github.com/python/cpython/blob/master/Lib/importlib/_bootstrap.py#L952
Shows loading parent first.

There's also a piece about infinite recursion.  If you import a child module in its `__init__.py`, wouldn't it enter an infinite loop?  I believe the solution was to add the package to the "modules" section before executing anything, that way if during the execution, the package was imported again, it would continue as normal.  However, this has the odd property that if you relied on something being done in `__init__.py`, and you also import that module, I don't think you can rely on it being done before you import it...  I'll want to test this out.

Yes, so the `__init__.py` can define the order of imports and this will be respected.  So if you have external effects or other dependencies, if you lay them out in order in your `__init__.py`,
they will always be respected.

When does it decide if it is a package?
Where to do the loaders come from? Sys.meta_path?

In my environment, meta_path, and path_hooks both can't handle finding 'al.marbury'.  There must be something else that looks at the "path" variable.



http://python-notes.curiousefficiency.org/en/latest/python_concepts/break_else.html


Could write a simpler python implementation in order to understand its internals
better.  Could write it in rust instead of C.  Could also just use C.

I also kind of want to make this space thing I've been talking about.
