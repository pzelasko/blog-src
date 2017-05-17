Title: Creating modules in Python
Lead: A few tips to help you avoid import errors
Published: 9/5/2017
Tags: 
    - Python
---

## Introduction

Regardless of whether you're building a web app, doing a data science project or just automating some workflow, you might find Python a very friendly language to do your work. Nonetheless, when I was starting to learn Python, I've experienced some struggles when it came to creating my own, easily importable modules and packages. Read further and check out a few tips which may save you the headache.

## Modules and packages

A Python module is just a single file with variables, functions and classes which can be imported. For example, if you have a file named `my_module.py`, you can import it by typing

    import my_module
    
in your script. 

On the other hand, a package is a directory with modules, which additionally has to contain an `__init__.py` file (which may or may not be empty - I might yet write about it). Seems easy? Rightfully so. But there's a catch to it - the Python interpreter has to be able to find your file. There are several ways to make it happen:
- the imported file must be present in the current working directory, or
- the directory containing the file must be present in the [PATH environment variable](https://en.wikipedia.org/wiki/PATH_(variable) ), or
- the package containing the module has to be installed.

The first option is easy enough, but you'll probably want to keep your files in more places than one directory. Let's see what can go wrong.

## The problem

Suppose we have a package with following structure:

    my_package
    |
    +-- first_subpackage
    |   |-- __init__.py
    |   +-- A.py
    +-- second_subpackage
    |   |-- __init__.py
    |   |-- B.py
    |   +-- holy_grail.py
    |-- __init__.py
    |-- C.py
    +-- D.py
    
As long as we're working within this package, there's no problem, we can `import C` inside `D.py`, as well as `import first_subpackage.A`. However, when we want to `import C` inside `A.py` or `import first_subpackage.A` inside `B.py`, we've got a problem:

    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    ImportError: No module named 'C'
    
There is a quick and dirty workaround - you can register the parent path during script execution in the PATH variable, by adding these lines in `A.py`:

    import sys
    sys.path.append('..')
    
The double dot means literally one directory up, i.e. the `my_package` directory. However, there's a better way to handle this.

## The solution

An installation script. You're probably familiar with the `pip` package manager - besides installing libraries from the [PyPI repository](https://pypi.python.org/pypi), it enables you to perform an installation of your own package. All you need to do is to prepare a `setup.py` script, to be placed one directory above the `my_package` directory. The `setup.py` script contains information about your package, such as name, author, etc., but most importantly - the layout of your package.

I won't get into much details now, but the basic form of this script would look like this:

    from distutils.core import setup
    setup(
        name='my_package',
        version='1.0',
        author='Brian',
        maintainer='Not Brian',
        description='Nice package.',
        py_modules=['my_package']
    )

In order to perform installation, you need to type this command in your terminal:

    path/to/my_package$ pip install -e .
    
This makes `pip` install the package in a development mode - it means that no files will be copied, and the installation will be updated each time you modify the code - great! Now you will be able to perform previous imports without any `ImportError`s. As a rule of thumb, inside `my_package`, you should always import by typing the full path to the imported module - e.g. in `B.py`, use `import my_package.second_subpackage.holy_grail` instead of `import holy_grail`. Additional benefit is that now you can `import my_package` using the Python interpreter regardless of which directory you started it in. 

## Summary

I've described some ways of handling Python modules and packages. Even though there are quick workarounds, the best option is to write a simple installation script. There's a lot more that can be said on this topic, but I hope you found this short intro helpful.
