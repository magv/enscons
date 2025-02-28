*******
enscons
*******

Build Python packages with SCons.

Enscons is a Python packaging tool based on `SCons <http://scons.org/>`_.  It builds pip-compatible source distributions and wheels without using distutils or setuptools, including distributions with C extensions.  Enscons has a different architecture and philosophy than distutils.  Rather than adding build features to a Python packaging system, enscons adds Python packaging to a general purpose build system.

Enscons helps you to build sdists that can be automatically built by pip, and wheels that are independent of enscons.

What does enscons provide?
--------------------------

Enscons provides a small amount of code to generate wheels, do development installs, generate egg-info and dist-info metadata, do basic conversion of setup.py to pyproject.toml and SConstruct, and configure SCons' compilers with the flags distutils would use to compile C extensions.  

Enscons implents a `PEP 518 <https://www.python.org/dev/peps/pep-0518/>`_ build backend, and a setup.py shim that can be used for `pip install -e .`

Enscons provides code to translate from the arguments pip and tox send to setup.py to something SCons can use, and then executes the SConstruct, the SCons analog to a Makefile. 

Why might you want to use enscons?
----------------------------------

Enscons is most useful if you want to use SCons.  SCons has superpowers that distutils doesn't.  You might want to use enscons if:

* You can never remember how to write MANIFEST.in.

  Enscons does not use MANIFEST.in.  Instead, SCons automatically knows which files are source based on which files you've told it to build or install.  You build this list with straightforward ``glob()``, or any Python code.

* You would like to include generated files in your sdist.

  All Python sdists include generated PKG-INFO metadata, but sometimes it is useful to include other generated files, such as generated C extension code, in a sdist.  Once SCons knows how to build your file, just appending it to the list of sources will cause it to be automatically rebuilt and included in your source archive.

* You sometimes forget to clean your build directory.

  SCons does not care if there are extra files in your build directory.  Its dependency tracking technology knows exactly the files you meant to build; if you build wheels for PyPy and CPython (different C extension filenames) without cleaning the build directory, it will only include the correct extensions in each wheel.

* You prefer to build only when necessary.

  SCons only rebuilds your C extensions, sdists, and wheels when their dependencies have changed.

* You need to extend distutils.

  The most compelling reason to use enscons is if distutils (or setuptools, which shares its architecture) is not meeting your needs.  Distutils' fundamental subclass-based architecture makes it notoriously difficult to extend, and any distutils extension must be debugged against several versions of Python, setuptools, and any other distutils extension your end users might be using.
  
  In SCons, adding a simple compiler is practically a one-liner.  If your project needs to use code generation or invoke any kind of compiler or build chain that is not supported by distutils, it is probably easier to switch than to extend distutils.


* You would like to develop your own distutils replacement.

  Enscons is well under 1,000 lines of code, but implements ``wheel``, ``setup.py``, ``setup.py develop``, and other tricky bits that can seem like magic.  The important parts of distutils are simple.  Enscons might be a useful reference for someone who would like to implement the same features on top of a different general-purpose build system.

Using enscons
=============

Install enscons with ``pip install enscons``.

If you are starting from an existing project, try running ``python -m enscons.setup2toml`` next to your ``setup.py`` to convert.  It will create a ``pyproject.toml`` and ``SConstruct`` that should serve as a good starting point.  It can convert simple pure-Python projects with only ``*.py`` but the ``SConstruct`` will require some tweaking for projects with non-Python files or C extensions.

Build the project by running ``python -m SCons``.

If you are not starting from an existing ``setup.py``, get enscons' source code, and copy ``pyproject.toml``, ``SConstruct``, and ``setup.py`` into your own project.

Edit ``pyproject.toml`` with your own metadata keys in `PEP 621 <https://www.python.org/dev/peps/pep-0621/>`_ format.
Enscons will use those keys to generate the static metadata files (``PKG-INFO`` and ``METADATA``).
In addition to PEP 621, some ``setup.py``-style keys are recognized: ``description_file`` (superceeded by ``readme``), ``url``, ``author`` and ``author_email`` (superceeded by ``authors``), ``license`` as a string, ``keywords`` as a string, ``platform``, ``install_requires`` (superceeded by ``dependencies``), ``extras_require`` (superceeded by ``optional-dependencies``), and ``src_root``.

Example enscons projects
------------------------

Right now, the best way to learn how enscons works is by example.

* `rsalette <https://github.com/dholth/rsalette/>`_ Simple package with just two modules.
* `cryptacular <https://github.com/dholth/cryptacular/>`_ Has a C extension.
* `pysdl2-cffi <https://github.com/dholth/pysdl2-cffi/>`_ Generates Python and C source code as part of the build, then compiles the generated source.
* `hello-pyrust <https://github.com/dholth/hello-pyrust>`_ An extension using `Rust <https://www.rust-lang.org/>`_ and `cffi <http://cffi.readthedocs.io/en/latest/>`_ instead of C.
* `enscons <https://github.com/dholth/enscons/>`_ Enscons builds itself.
* `nonstdlib <https://github.com/dholth/nonstdlib/>`_ Experimental repackaging of the Python standard library into many wheels.


More about SCons
================

SCons is a general purpose build system. It is pure Python, it can run on both Python 2 and 3, and it has a good community.  Very few other build systems meet all of these criteria.  These features make SCons a good platform for an experimental Python packaging system.
