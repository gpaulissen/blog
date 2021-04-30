---
title: Use (B)ridge (M)arkup (L)anguage with Docker
categories: [ development, bridge ]
tags: [ bml, docker, LaTeX ]
permalink: /bml-docker/
classes: wide
toc: true
toc_label: "Table of contents"
---

This post describes how BML is used in
combination with [Docker](https://docs.docker.com/).

# Introduction

The purpose of the Bridge Bidding Markup Language (BML) is to offer an easy
way of documenting contract bridge bidding systems. The file(s) created are
supposed to be easy to read for both human and machines.

Please have a look at the [BML User Guide](https://gpaulissen.github.io/bml/)
for more information.

You can see the code here: [GitHub BML
project](https://github.com/gpaulissen/bml).

Please note that this is not a Docker tutorial.

BML is installed with utilities that need Python to run. And one of the
utilities creates LaTeX files which need to be processed by a LaTeX program in
combination with Perl. And there is a Makefile distributed with the BML source
in order to facilitate running those programs but that needs make. Since
installing all those programs may be difficult for a user, I thought: "Why not
Dockerize it?".

That may also serve as good starting point for a web site where you can upload
your BML file(s) and receive the generated files. Something like that is ideal
using Node.js in combination with Docker.

# Analysis

The Docker image can be based on
[node:alpine](https://github.com/nodejs/docker-node#nodealpine) since that is
a minimal image already containing Node.js.

So the programs we need are:
- Perl
- LaTeX
- Make
- Python (and pip)
- BML utilities

## Alpine Linux

Thanks to the [Alpine Linux package
repository](https://pkgs.alpinelinux.org/packages) it was easy to find the
package names for the programs.

The Alpine Linux package installer is called `apk`.

## Perl

The command `apk perl` will add perl.

## LaTeX

In order to keep the Docker image small I just want to install the
LaTeX packages needed for BML. Well, that is feasible since
the expected BML test TeX files contain already all the possible TeX
`\usepackage{}` directives.

So the idea is to create PDF files from those
expected test TeX files after the minimal installation just to verify that
everything is there.

There are several LaTeX distributions like TeX Live and MiKTeX. Since MiKTeX
is fairly small and has a Docker image I tried that first but I could not get
it working.

After some searching I found [TinyTeX](https://yihui.org/tinytex/), even
smaller than MiKTeX. And very simple to install. In fact you have to download
a script that will do the rest of the installation:

```
wget -qO- "https://yihui.org/tinytex/install-unx.sh" | sh -s - --admin --no-path
```

Since wget is not installed by default the command `apk wget fontconfig
freetype gnupg` will install the prerequisites for TinyTeX.

## Make

The command `apk make` will add make.

## Python

The command `apk python3 py3-pip py3-setuptools` will install:

- python 3
- pip 3
- setuptools (needed to install BML on Docker)

## BML utilities

The command `pip install bridge-markup` should do the trick. Can only be done
after `python` and `setuptools` have been installed.

# Implementation

You should try to minimize the number of layers so one must try to keep the
number of commands small by chaining commands (cmd1 && cmd2 && ...) if possible.

## BML Dockerfile

This is the (simplified) BML Dockerfile (see also the [reference](https://docs.docker.com/engine/reference/builder/)):

```
FROM node:alpine

LABEL Description="Dockerized BML" Vendor="Gert-Jan Paulissen"

# 1) Install:
#    - perl, wget and libraries needed for TinyTeX (https://yihui.org/tinytex/)
#    - make
#    - python3
#    - su-exec
# 2) Create user bml and show its details
RUN apk update && apk upgrade && apk add --no-cache perl wget fontconfig freetype gnupg make python3 py3-pip py3-setuptools su-exec &&\
    addgroup -S bml && adduser -S -G bml bml && getent passwd bml

# This is bml root directory
WORKDIR /bml
  
COPY . .

# Install TinyTeX as bml
USER bml

# Prepare to install TinyTeX system wide (see https://yihui.org/tinytex/faq/)
RUN wget -qO- "https://yihui.org/tinytex/install-unx.sh" | sh -s - --admin --no-path

# Back to root to continue with some tasks
USER root

# 1) install TinyTeX in /usr/local/bin
# 2) install BML and test that the executables are there
# 3) modify permissions for /bml
RUN ~bml/.TinyTeX/bin/*/tlmgr path add &&\
    pip3 install -e . && which bml2bss bml2html bml2latex bss2bml bml_makedepend &&\
    chown -R bml:bml . && chmod -R 755 .
    
USER bml

# 1) Install missing LaTeX packages (by looking at errors from next statement)
# 2) Generate some PDFs to test a complete LaTeX installation
RUN tlmgr install dirtree listliketab parskip pbox txfonts &&\
    latexmk -pdf -output-directory=/tmp /bml/test/expected/example.tex /bml/test/expected/example-tree.tex

ENTRYPOINT ["/bml/entrypoint.sh"]

# This is the place where input files are expected and where we run make from
WORKDIR /bml/files

# Default command
CMD ["make", "-f", "/bml/bml.mk", "help"]
```

## First RUN

The first RUN command installs (as user root) perl, make, python3 and su-exec. The
latter is a utility to run commands as another user when one is logged in as
root.

And this RUN creates a group and user bml that should run the container.

## Second RUN

As user bml get the TinyTeX shell script and prepare to install it system wide.

## Third RUN

Back to user root again to:
1. install TinyTeX in /usr/local/bin
2. install BML and test that the executables are there
3. modify permissions for /bml

## Fourth (and last) RUN

As user bml:
1. install missing LaTeX packages
2. generate some PDFs to test a complete LaTeX installation

We are now sure that LaTeX works for any generated TeX file since all the
possible variations of LaTeX packages needed have been tested.

## ENTRYPOINT and CMD

The ENTRYPOINT and CMD:
- ENTRYPOINT ["/bml/entrypoint.sh"]
- CMD ["make", "-f", "/bml/bml.mk", "help"]

This means that by default the container will start with:

```
/bml/entrypoint.sh make -f /bml/bml.mk help
```

You can not (or better, almost not) change the ENTRYPOINT, but you can easily change the
CMD allowing you to do whatever needed on the container.

Best practices prescribe not to run the container as root so I have created a user bml that
will run the container, but there may be situations where you need to run as
another user (nor root, nor bml). In that case you start the container as
root first with the wanted user id and group id passed as environment variables:

```
docker run -u 0:0 -e UID=<uid wanted> -e GID:<gid wanted> ...
```

Please note that `-u 0:0` means that the container will start with user id (uid) 0 and
group id (gid) 0, which is root.

This is the `entrypoint.sh` script:

```
#!/bin/sh

test -z "$DEBUG" || { id && set -x; }

# current user and group id
uid=`id -u`
gid=`id -g`

# wanted user and group id (default to current)
UID=${UID:-$uid}
GID=${GID:-$gid}

# user and group id of current user equal to wanted user?
if [ "$uid" -eq "$UID" -a "$gid" -eq "$GID" ]
then
    # just spawn the command
    exec "$@"   
else
    # must create a host user and group, add it to group bml (owner of software) and spawn if that succeeds
    addgroup --gid $GID host_user && \
        adduser --disabled-password --gecos "" --ingroup host_user --no-create-home --uid $UID host_user && \
        adduser host_user bml && \
        exec su-exec host_user "$@"
fi
```

If the current and wanted user (and group) are the same then the command is
just spawned as the current user. If not, the wanted user is created with name
`host_user` and the command is spawned by that user. Please note that su-exec
will only work as the current user is root.

## WORKDIR

The container working directory for BML is /bml/files. This directory can
be mapped to a Docker volume or a bind mount, a file or directory on the *host
machine* that is mounted into a container. This setup allows us to generate
BML output on a host machine using a Docker container.

# Usage

The following command creates a file test/data/example.pdf from the
test/data/example.bml file:

```
make -f docker.mk run CMD="make -f ../bml.mk example.pdf"
```

Please note that by default the container /bml/files is mapped to
the (full path of the) test/data folder as indicated by the make variable BML_FILES. 

For more help just run:

```
make -f docker.mk run
```

# Conclusion

I hope that this simple Docker guide has give you some appetite. And I must
say that I was happy to combine new (Docker) and old (make, LaTeX) software to
create something that I can use for the next step: creating a web interface
for BML.

Stay tuned!
