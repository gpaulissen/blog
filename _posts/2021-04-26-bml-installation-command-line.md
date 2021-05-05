---
title: Install (B)ridge (M)arkup (L)anguage using the Command Line
categories: bridge
tags: [ bml ]
permalink: /bml-installation-command-line/
classes: wide
toc: true
toc_label: "Table of contents"
---

This is the guide for installing Bridge Markup Language using the Command Line on:
- Unix
- Linux
- Mac OS X
- Windows
- Cygwin on Windows
- Windows Subsystem for Linux

This guide is meant for the more technically experienced people like
developers or power users.

# Install Python

You need Python version 3 (or higher) in order to use the BML converters. Get it
from <http://www.python.org> or, even better, use
[Miniconda](https://docs.conda.io/en/latest/miniconda.html) so you can have
easily several versions of a Python environment.

Verify you installation by starting a command prompt and issue these commands:

```
python --version
pip --version
```

# Install BML

## Installation using PyPI

To install the BML converters, issue this command from a command line:

```
pip install bridge-markup --upgrade
```

## Installation from source

Another possibility when you want to develop or inspect the sources from the
GitHub repository:

### Download and install

Issue these commands from a command line:

```
git clone https://github.com/gpaulissen/bml.git
pip install -e .
```

### Test

To run the tests from the development version you can use the py.test command:

```
py.test
```

You may need to install the required test packages first by this command:

```
pip install -r test_requirements.txt
```

## Test the installation of BML

Issue these commands to verify the installation:

```
bml2bss --version
bml2html --version
bml2latex --version
```

# Install LaTeX

In order to create a PDF from a BML file you need to:
1. generate a TeX file with the bml2latex converter
2. generate a PDF from TeX file using a LaTeX program

To install a basic TeX/LaTex program go to [MiKTeX download](https://miktex.org/download) and follow the instructions.

## Test the installation of LaTeX

Issue this command to verify the installation:

```
latexmk -h
```
