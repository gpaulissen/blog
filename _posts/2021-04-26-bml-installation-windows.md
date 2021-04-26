---
title: Install (B)ridge (M)arkup (L)anguage on Windows
categories: bridge
tags: [ bml ]
permalink: /bml-installation-windows/
classes: wide
toc: true
toc_label: "Table of contents"
---

This is the guide for installing Bridge Markup Language on Windows 10. It is
intended for common end users who prefer to use a graphical user interface (GUI).

# Open a Windows Command Line

To install or verify the programs installed, you need to open a Windows
Command Line.

This section explains how to do that since you will need it several times in the rest of this document.

## Using the Windows+R keys

Please note that to open a Windows Command Line you can press the Windows
key together with the R key to open 'Run' box. Then type 'cmd' and then click
'OK' to open a regular Command Line.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/windows-r.png" alt="Start Command Prompt using Windows+R keys">*

## Using the Windows status bar

You may also enter `command` in the Windows status bar search box and click the Command Prompt App.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/search-command.png" alt="Start Command Prompt using Windows search bar">

# Install Python

## Using the Windows Store

Type `store` in the Windows status bar search box:

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/search-windows-store.png" alt="Start Microsoft Store">

Click on the Microsoft Store App.

Next type `python` in the Microsoft Store search box and press ENTER.

You will see a list of Python versions:

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/windows-store-1.png" alt="Search Python in Microsoft Store">

Next choose the highest version to get, click on its icon and click on `Get` to get it:

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/windows-store-2.png" alt="Get Python in Microsoft Store">

Now you may see something like this, so click on `Install`:

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/windows-store-3.png" alt="Get Python in Microsoft Store">

Next click on `Install on my devices`:

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/windows-store-4.png" alt="Get Python in Microsoft Store">

## Using the Python home page

You can also install Python by going to the [Python home
page](https://www.python.org/):

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/python-home-page.png" alt="Python home page">

Click on the Latest version in the `Download` section and choose the Windows installer
(64 bit usually):

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/python-download-page.png" alt="Python download page">

Next click on the downloaded executable and this window will show up:

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/python-installer.png" alt="Run Python installer">

Please click on **Add Python 3.9 to PATH** before pressing `Install Now`.

## Test the installation of Python

Issue these commands from a command line to verify the installation:

```
python --version
pip --version
```

# Install BML

## Go to PyPI

**You may skip this section since it is purely informational**.

The BML converters are available as a PyPI (Python Package Index) package.

Go to [PyPI](https://pypi.org/) and search for `bridge-markup`:

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/pypi-bridge-markup-1.png" alt="PyPI home page">

Choose the correct project (the first one here):

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/pypi-bridge-markup-2.png" alt="Search PyPI for bridge-markup">

Next, just copy the text `pip install bridge-markup` to the clipboard as shown below:

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/pypi-bridge-markup-3.png" alt="Copy text to clipboard">

Do **NOT** use `Download files` here. The text you copied will be used below.

## Use the Windows Command Line

To install the BML converters, issue this command from a command line:

```
pip install bridge-markup
```

You can also paste the text copied from PyPI (press ENTER after the paste):

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/windows-command-line-paste.png" alt="Install bridge-markup using pip">

## Test the installation of BML

Issue these commands from a command line to verify the installation of the converters:

```
bml2bss -h
bml2html -h
bml2latex -h
```

# Install LaTeX

In order to create a PDF from a BML file you need to:
1. generate a TeX file with the bml2latex converter
2. generate a PDF from TeX file using a LaTeX program

To install a basic TeX/LaTex program on Windows go to [MiKTeX download](https://miktex.org/download):

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/miktex-download-page.png" alt="MiKTeX download page">

Download the executable and run it (choose **Install MiKTeX only for me** later on):

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/miktex-installer.png" alt="Run MiKTex installer">

## Test the installation of LaTeX

Issue this command from a command line to verify the installation:

```
latexmk -h
```
