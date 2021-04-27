---
title: Use (B)ridge (M)arkup (L)anguage with Docker
categories: [ development, bridge ]
tags: [ bml, docker ]
permalink: /bml-docker/
classes: wide
toc: true
toc_label: "Table of contents"
---

This post describes how BML is used in combination with Docker.

# Introduction

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

So the programs we need are:
- Python (and pip)
- LaTeX
- Perl
- Make
- BML utilities

The image can be based on
[node:alpine](https://github.com/nodejs/docker-node#nodealpine) since that is
a minimal image already containing Node.js.

## Python

`apt-get python` should do the trick.

You should try to minimize the number of layers so one must use one Docker RUN to
install all standard executables like Python, LaTeX, Perl and Make.

## LaTeX

There are several LaTeX distributions like TeX Live and MiKTeX. I want the
smallest distribution possible so I will choose MiKTeX since that allows you
to install packages on the fly unlike TeX Live. But I do not want that
packages are added after the image is created so I need to install the
packages needed for BML during the docker build. Well, that is feasible since
the expected BML test TeX files contain already all the possible TeX
\usepackage{} directives. So the idea is to create PDF files from those
expected test TeX files and then do not allow anymore an installation of
update usig the MiKTeX package manager. The initial setup should run as root
so that after adding a Docker USER directive nothing can be changed anymore
which is good from a security perspective.

See below for the MiKTeX Dockerfile.

## Perl

Add `apt-get perl` to the RUN command.

## Make

Add `apt-get make` to the RUN command.

## BML utilities

`pip install bridge-markup` should do the trick. Can only be done after python
has been installed hence a new RUN command.

# Implementation

## Current MiKTeX Dockerfile

This is the [MiKTeX Dockerfile](https://github.com/MiKTeX/docker-miktex/blob/master/Dockerfile):


```
FROM ubuntu:bionic

LABEL Description="Dockerized MiKTeX, Ubuntu 20.04" Vendor="Christian Schenk" Version="20.7"

RUN    apt-get update \
    && apt-get install -y --no-install-recommends \
           apt-transport-https \
           ca-certificates \
           dirmngr \
           ghostscript \
           gnupg \
           gosu \
           make \
           perl

RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys D6BC243565B2087BC3F897C9277A7293F59E4889
RUN echo "deb http://miktex.org/download/ubuntu focal universe" | tee /etc/apt/sources.list.d/miktex.list

RUN    apt-get update \
    && apt-get install -y --no-install-recommends \
           miktex

RUN    miktexsetup finish \
    && initexmf --admin --set-config-value=[MPM]AutoInstall=1 \
    && mpm --admin --update-db \
    && mpm --admin \
           --install amsfonts \
           --install biber-linux-x86_64 \
    && initexmf --admin --update-fndb

COPY entrypoint.sh /
ENTRYPOINT ["/entrypoint.sh"]

ENV MIKTEX_USERCONFIG=/miktex/.miktex/texmfs/config
ENV MIKTEX_USERDATA=/miktex/.miktex/texmfs/data
ENV MIKTEX_USERINSTALL=/miktex/.miktex/texmfs/install

WORKDIR /miktex/work

CMD ["bash"]
```

## Base image

That will become miktex/miktex:latest
