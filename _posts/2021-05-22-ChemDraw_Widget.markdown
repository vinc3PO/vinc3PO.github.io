---
layout: post
title:  "A ChemDraw widget for PyQt5"
date:   2021-05-22 12:33:52 +0200
---

A few years back, I had the idea to create an inventory system. I decided, for now obscure reasons, to go for a local application using PyQt5 as a graphical interface.

One of these reasons was the possibility to create a ChemDraw activeX widget from which I would be able to get information regarding the drawn molecule.

<br/><br/>
# CLSID and PyQt5.QAxContainer

The CLSID is a unique identifier of a com object and can be found in the registry. 

`HKEY_LOCAL_MACHINE\SOFTWARE\Classes\ChemDrawControlXX.ChemDrawCtl\CLSID\` -> XX being the version (ie 20)

For ChemOffice 19 the CLSID was something like `84328ED3-9299-409F-8FCC-6BB1BE585D08`

With the CLSID in hand, we can now easily create a widget.

{% gist bb4492b16358fca85dbc11cacfc16955 basic_chemdraw.py %}

Et voila, this is how to create a ChemDraw widget.

**The python version must match that of ChemDraw, as a 64 bits version of python would not be able to load the 32 bits of ChemOffice.**

<br/><br/>
# Dealing with updates and versions

As a unique identifier doing its unique job of being unique, the CLSID changes with a new version. I have built a little function that does it for me. The `get_clsid` will return the list of clsid of _chemdrawctl_ order by date. 

{% gist bb4492b16358fca85dbc11cacfc16955 clsid_finder.py %}


<br/><br/>
# Subclassing QAxWidget()

In order to give more control to the widget, subclassing of QAxWidget is the way to go. 

{% gist bb4492b16358fca85dbc11cacfc16955 chemdraw_class.py %}

In this class, the widget is initialised using the `get_clsid()` function, meaning that is ChemOffice is updated the widget will automatically find a valid CLSID.

<br/><br/>
# Get activeX control info

In the class below, you can see an example possible of commands. PyQt5 tool `dumpdoc.exe` allows quick access to the documentation of the activeX control. You might need to install `PyQt5-tools` to have it.

```dumpdoc.exe -o PATH_TO_FILE.html ChemDrawControlXX.ChemDrawCtl``` 

where XX is your ChemOffice version (e.g. 18, 19).

If you have trouble finding the _dumpdoc.exe_, you may want to look there:

`YOUR_PYTHON_FOLDER\Lib\site-packages\qt5_applications\Qt\bin`

<br/><br/>
# PySide2 vs PyQt5
Both can handle activeX control, have a fairly similar semantic but have different licensing (GPL vs LGPL). You can look it up online if you are up to this kind of things.

However, the implementation of the dynamic call might diverge, and some command might even not be available.

<br/><br/>
# Wrap-up
Here is one possibility to incorporate ChemDraw activeX widget within your application. This an easy way to get chemical information from a structure such as smiles/png/... and make is easier to copy/paste from other PerkinElmer products (`Signals Notebook`, `ChemFinder`, etc).

However, there is a huge drawback to this method. PyQt or Qt in general doesn't allow graphical widget to be loaded outside of the main thread and ChemDraw is not the fastest to load. So at start of the application/widget, the loading time (a few seconds) will let you time to rethink your life choices.