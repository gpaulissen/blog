---
title: "Jython for Robotframework EDitor"
categories: testing
tags: [ jython, red, ant, perl, eclipse, robotframework ]
permalink: /jython-red/
---
Setup of Jython for RED, the Robot Framework editor for Eclipse

# Introduction

When you use RED, you can use Python with Robot Framework and all its
installed libraries (Selenium for instance) to launch tests. However, if you
use Maven and thus the Maven Robot Framework plugin, you risk to use a
different Robot Framework version with RED than you use with Maven. Plus, the
Python variant in RED cannot use Java libraries like the Maven variant does
(which is based on Jython). You may use Jython of course but the last versions
2.7.0 and 2.7.1 did not run well on my Windows 7 laptop due to Pip and DLL
problems respectively. And Jython seems to be not so well supported as Python
is.

But there seemed to be a solution,
[How to integrate RED with Robotframework Maven Plugin](http://nokia.github.io/RED/help/user_guide/tools_integration/maven.html),
that describes a way to use the same Robot Framework jar as Maven does. In
that solution the launch4j program is used to create a Jython executable based
on the Robot Framework Jar (which includes Jython as well). There are a few
snags however as I discovered when I tried to use that approach.

This Blog is written for a Windows environment (it has been tested on Windows
7 and Windows 10), but it might also work on a Unix like environment.

# Problems encountered

## Classpath entries

First, you have to hard code your Jar libraries as Java classpath entries into
the launch4j configuration file. That means that if you need a new Jar library
you would need to recreate the Jython executable. And I needed not only Robot
Framework, Selenium but also a JDBC database library and a JMS library. And
other projects may have other constraints. Actually what you would like to use
is the compile classpath of Maven which is a complete list of jars and its
dependencies.

## Jython arguments

The second problem was that RED launches the generated Jython program with
arguments not recognized by the robotframework.jar (for instance -J-cp
&lt;classpath&gt;) or with arguments just applicable for a Java interpreter
like -Dvar=value.

## Java version

The third and last problem was that launch4j searches the Java executable a
little bit too sophisticated (or complicated): it uses the Windows registry to
look for a system-wide installed JRE/JDK with a minimum and/or maximum
version. On the Windows 7 system I was using, only Java 1.7 was installed
system-wide and I had no privileges to install a higher version. And the
Selenium library really needed Java 1.8. I was stuck.

# A workaround

So what to do?

The first and second problem are actually environment problems. I need
another program to set up an environment (for instance set environment
variables, determine the Maven classpath and change the command line) before
launching the robotframework.jar with arguments it recognizes. My old friend,
script language Perl, came to my rescue. The Windows variant Strawberry Perl
allows you to create executables based on Perl scripts using the Perl CPAN
module Par::packer.

So the idea is to create an executable named Jython (jython.exe) that is
called from the Eclipse RED editor and that sets up the environment before
launching Java with the robotframework.jar.

All the build steps must be automated as well of course and what is more easy
to use than good old Ant? I really have no idea.

# Installation

See the [installation manual](https://gpaulissen.github.io/jython/INSTALLATION.html).
