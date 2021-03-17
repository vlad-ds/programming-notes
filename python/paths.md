# Python paths.



SOURCE: https://realpython.com/python-import/

**Module**. An object that serves as an organizational unit of Python code. Modules have a namespace containing arbitrary Python objects. Modules are loaded into Python by the process of importing. 

**Package**. A Python module which can contain submodules or recursively, subpackages. Technically, a package is a Python module with an `__path__` attribute.

In practice, a package is a directory containing Python files and other directories. It also contains a file named `__init__.py`. Directories without this file are treated as **namespace packages**. 

In general, submodules and subpackages aren’t imported when you import a package. However, you can use `__init__.py` to include any or all submodules and subpackages if you want. (See example with `world` module!)

The module namespace is implemented as a dict and is available as `math.__dict__`. The global namespace is also a dict and can be accessed through `globals()`. 

----

`dir()` lists all the contents of a namespace. 

`dir(module)` lists all the contents of a module or package namespace.

##### Imports

`import math as m` 

`from math import pi as PI`

`from . import africa`  relative import: from the current package, import the subpackage Africa

-------

Python looks for modules and packages in its **import path**. You can see this path by printing `sys.path`. There are 3 different kinds of locations:

1. Directory of the current script 
2. Contents of the PYTHONPATH environment variable
3. Other, installation-dependent directories

"Typically, Python will start at the beginning of the list of locations and look for a given module in each location until the first match. Since the script directory or the current directory is always first in this list, you can make sure that your scripts find your self-made modules and packages by organizing your directories and being careful about which directory you run Python from."

Be careful not to create modules that **shadow** other important modules. 

----

#### Create a local package

When you install a package locally, it's available to all scripts in the environment. 

Create `setup.cfg` and `setup.py` files. 

It is recommended to give them a common prefix like`_local`. To install the package locally:

```python -m pip install -e .```

You can also change your import path. Given two folders `third_party` and `local`, you can do: 

```python
import sys
sys.path.extend(["third_party", "local"])
```

To make the packages available. 

-----

#### Import Style Guide

- Keep imports at the top of the file.
- Write imports on separate lines.
- Organize imports into groups: first standard library imports, then third-party imports, and finally local application or library imports.
- Order imports alphabetically within each group.
- Prefer absolute imports over relative imports.
- Avoid wildcard imports like `from module import *`.

-----------

#### Resource Imports (!)

`importlib.resources` gives access to resources within packages. Part of the standard library. 

"If you can import a package, you can access resources within that package."

To open the `alice_in_wonderland.txt` resource in the `books` directory (which has an `__init__.py` file):

```python
from importlib import resources
with resources.open_text("books", "alice_in_wonderland.txt") as fid:
	alice = fid.readlines()
```

"When distributing your package, you’re not even guaranteed that resource files will exist as physical files on the file system. `importlib.resources` solves this by providing `path()`. This function will return a [path](https://realpython.com/python-pathlib/) to the resource file, creating a temporary file if necessary."

To make sure any temporary files are cleaned up properly, you should use `path()` as a context manager using the keyword `with`:

```python
from importlib import resources
with resources.path("hello_gui.gui_resources", "logo.png") as path:
	print(path)
```

Given the file hierarchy:

```
hello_gui/
│
├── gui_resources/
│   ├── __init__.py
│   ├── hand.png
│   └── logo.png
│
└── __main__.py
```