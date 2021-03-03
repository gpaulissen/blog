---
title: How to build an Oracle APEX application (1)
categories: development
tags: [ Oracle, APEX, DevOps, DataModeling ]
permalink: /oracle-apex-how-to-build-1/
toc: true
toc_label: "Table of contents"
toc_icon: "database"
excerpt: "An introduction on \"How to build an Oracle APEX application\"."
last_modified_at: 2021-03-03T15:20:00
---

<figure class="centered">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/512px-Devops-toolchain.svg.png" alt="">
	<figcaption>Kharnagy, CC BY-SA 4.0 <https://creativecommons.org/licenses/by-sa/4.0>, via Wikimedia Commons</figcaption>
</figure> 

# Why this subject and why now?

Last week a former manager of mine complimented me on [my LinkedIn
account](https://www.linkedin.com/in/gert-jan-paulissen-2508203) for a Blog
called [About Oracle apex and translations
(2)](https://www.linkedin.com/pulse/oracle-apex-translations-2-gert-jan-paulissen/?trackingId=dpjSfVX%2FeD6KCXJMF1OHXw%3D%3D). His
message was that is good to share knowledge as he is doing himself now
too. And yeah, I whole-heartedly agree. And when my current boss wanted me to
write about how I build Oracle APEX applications from the beginning till the
end, I thought let's do it before I leave the company. The funny thing is that
I have been introduced to my current boss by another former manager that
shares the same first name as the other manager. Thanks Harm I and II, for
your gentle words. Thank you boss for pushing me to write about "How to build
an Oracle APEX application".

Already soon in my career I invented solutions for not installing applications
manually. Sometimes the boss/manager/team did not see a value added right away
but after some time they got convinced, well almost all of them. In the Oracle
Designer era, I repeated this while working on an assignment for the ING bank
in Amsterdam. And in 2015 when I was working for pension fund MN in The Hague,
The Netherlands, I joined the Continuous Integration team and started to
assemble the ideas I am going to present to you here.

Actually I think I have enough material for a book or maybe even more. But
let's just start with a Blog and see what's comes of it. I won't dive too much
into details but I assure you that with the help of my ideas you are better
prepared to build a serious Oracle APEX application. And you can always
contact or hire me if you need more explanation :).

# What's in a name?

A title is important and I hope that it describes well what I do want to share
with you in this series of articles. It is not so much about how to use the
back-end part (Oracle Database) or the front-end (Oracle APEX). It is much
more about the tools, techniques and best practices around them in order to
build, deploy and maintain an Oracle APEX application efficiently and
correctly. Actually, APEX can be replaced by another front-end like Java ADF,
React, Angular or whatever.

I have used the word **build** on purpose and not
something like develop because I see an analogy with building a house. You
don't build a house by just buying parts like a door and some tools. No you need a
**plan**, an **architecture** if you prefer. And how often I see people
beginning with creating a table, an APEX screen and then they think they are
doing well. Maybe their boss/client is happy because s/he sees something visible but
IMO they just started without a plan. You just **DON'T** start with a door and
some tools when you need to build a house, so do not make the same
**mistake** when you build an Oracle application.

And there are other build analogies:
- the tool [Ant](https://ant.apache.org/) needs a build file to execute tasks;
- the Unix programs are usually installed after they are being built from source.

# Philosophy

I believe very much in the Unix approach of handling tasks. In Unix there are
a lot of simple tools that each in their own perform their task very well. I
like that approach so much because it enables you to use the same tools over
and over again. A win-win for you and your boss or client. Of course you may
decide to replace a tool but the idea is clear. And to be honest I do not like
it to learn every year another methodology or tool. I advance but not too fast
but not too late either. I assume that you do not rebuild your house every time there
are new techniques.

An example of this Unix approach is deploying your Oracle application
software. I use [Flyway](https://flywaydb.org/documentation/getstarted/why)
because that is a tool based on a simple idea: it executes database migration
scripts automatically and stores the result of the execution in a history table
so Flyway knows what has been installed in order to determine what to install
the next time. It is even not necessary to install version N+1 first if the
latest version installed was N, you can immediately continue with N+2 (or N+3
or ...). Simple and predictable, so I see no reason to use another tool. I
hope this convinces you to **never** again execute database migration scripts
**manually**.

Of course I have looked at the Supporting Objects feature of APEX but I think
it is only suitable for (demo) applications with a small number of
database objects that do no change. As soon as you build a real application
you will create a lot more database objects like packages and views and it
becomes too difficult to use APEX Supporting Objects, at least that is my
opinion. And do not forget that APEX is just the front-end so if you 
decide to replace it by another front-end you also have to find another tool
to run the migration scripts.

So embrace the Unix philosophy and use Flyway to run database migration scripts.

Another important point is to use the power of the Oracle Database. It is an
expensive product but very powerful so use it thoroughly and get used to it. Take
lessons, courses, read books, read Blogs but **invest** in it. It will really help
you to build better.

The last point is that we should be very vigilant regarding security, so just
apply all the best practices there are.

# Standing on the shoulders of giants

> If I have seen further it is by standing on the shoulders of Giants.
>
> <cite><a href="https://en.wikipedia.org/wiki/Standing_on_the_shoulders_of_giants">Isaac Newton in 1675</a></cite>

I have to mention Oracle gurus like Tom Kyte and Steven Feuerstein
but I should surely mention a fellow Dutchman Rob de Wijk who has written
Blogs about [implementing business rules](http://rwijk.blogspot.nl/2008/07/implementing-business-rules.html) in
2008 and [Professional Software Development using Oracle Application Express](http://www.rwijk.nl/AboutOracle/psdua.pdf) in 2013.
We are living in 2021 now and things have advanced but I have used his ideas to lay the foundation.

# Architecture

## Database structure

As always a picture is worth a thousand words so I will show the picture first:

<figure>
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/schema-structure.png" alt="">
	<figcaption>Schema structure by Rob van Wijk</figcaption>
</figure>

These layers are schemas in the database and folders in our application project environment.

Quoting Rob van Wijk:

> This layered approach is a choice we've made to
> enhance security and flexibility in our applications. The three schemas only have a minimal
> set of system privileges, just enough to create the object types needed for that layer.

### DATA

This is the schema that contains the data: the tables and all objects needed
to maintain the data logic. You may decide to put data logic packages in the
API layer but that is up to you.

The schema structure allows the UI layer to use objects from the DATA layer
but I think that should be only allowed for read access to simple tables,
think of List Of Values. All business logic should go via the API layer. It is
simple to define a view or package in the API layer that can be used for DML
purposes.

### API

This is the schema that contains the business logic. It may contain data logic
packages if you do not want to have packages in the data layer.

### UI

All User Interface logic. This means that this schema will be the parsing
schema for APEX. Please note that you can have more than one parsing schema
per APEX workspace so there is no problem having several applications with
different parsing schemas in a workspace.

### EXT

This is an EXTernal layer I have added to the structure. It is meant for
external logic: interfaces or data conversions. Please note that setting up a
new system almost always requires you to import data from another source. This
layer can take care of that. If your application has to support several customers
you may even have a layer for each customer. The level of this layer is the
same as the API layer. It can interact with the API layer in a bidirectional
way. After all, the external layer may use business logic and business logic
may use an interface from this layer. The UI layer may use objects from this layer too.

## Tools, techniques and best practices

### Oracle Database and Oracle APEX

I have used [Virtualbox](https://www.virtualbox.org/) and the prebuilt virtual
machine [Database App Development
VM](https://www.oracle.com/downloads/developer-vm/community-downloads.html)
from Oracle for my development environment. I strongly believe in having a
separate database for each developer while developing. I do not want to
interfere with others and I do not want that others interfere with me while I
work. At a later stage you can always use an integration or test database to
see if everything works well together.

Keep in mind that you cannot import an APEX application into another APEX
instance if the exported version is **higher** than the version of APEX to
import into. So exporting an APEX 19.2 application will **not** import into
APEX 18.2. So align all your APEX versions from development till production.
{: .notice--warning}

### Oracle SQL Developer Data Modeler

Maybe lesser known than its big brother Oracle SQL Developer but a tool that
allows you to build a great model (plan) of your database application. You can
use various modeling techniques like Entity Relationship Modeling and a lot, lot
more. It even allows you to create database scripts or migration scripts that
you may use in Flyway.

A book I can recommend is [Oracle SQL Developer Data Modeler for Database Design Mastery](https://www.goodreads.com/book/show/23871562-oracle-sql-developer-data-modeler-for-database-design-mastery) by [Heli Helskyaho](https://helifromfinland.blog/).

You can better use **one** modeling project for all your applications when you use SQL Data Modeler so you can share your configuration more easily between projects and developers.
{: .notice--warning}

### Version control

This is absolutely necessary IMHO. Use whatever tool you like, Git or
Subversion for instance, but **use** it. How often I needed to compare a
script with an older version I can not tell you, but it was often. And
sometime I had to just to throw away a concept to start over. And that is just
a small part of the advantages of a version tool. When you work in a team it
is a `sine qua non`.

SQL Data Modeler only supports Subversion but sites like [GitHub](https://docs.github.com/en/github/importing-your-projects-to-github/working-with-subversion-on-github) support both Git and Subversion.
{: .notice--info}

### Maven

[Apache Maven](https://maven.apache.org/index.html) is a software project
management and comprehension tool. Based on the concept of a project object
model (POM), Maven can manage a project's build, reporting and documentation
from a central piece of information.

So Maven will be the tool to automate several tasks like running Flyway,
exporting and importing APEX applications or running unit tests.

### Flyway

Already described, integrates very well with the tools above.

### Oracle SQL Developer

> Oracle SQL Developer is a free, integrated development environment that simplifies the development and management of Oracle Database in both traditional and Cloud deployments.
> SQL Developer offers complete end-to-end development of your PL/SQL applications,
> a worksheet for running queries and scripts,
> a DBA console for managing the database,
> a reports interface,
> a complete data modeling solution,
> and a migration platform for moving your 3rd party databases to Oracle.
>
> <cite><a href="https://www.oracle.com/database/technologies/appdev/sqldeveloper-landing.html">Oracle SQL Developer</a></cite>

So this tool is already a great asset for a database developer but it is absolutely
necessary when your DBA only allows you to access this tool and Java in a
Citrix environment where the command line or Maven is forbidden. After all,
Maven is just launching Java with some command line options. And you can
launch a program from the SQL Developer External Tools.

### utPLSQL

A PL/SQL unit testing framework originally developed by Steven Feuerstein, we
now have [version 3](http://utplsql.org/utPLSQL/latest/). An impressive piece
of work and easy to use. In the Java community it is normal to unit test but
not so in the Oracle community. This tool may convince you!

### SonarQube

A tool that might help with PL/SQL static code analysis is
[SonarQube](https://www.sonarqube.org/features/multi-languages/plsql/). Used
in combination with utPLSQL this tool will improve the quality of your
application code. Please note that this tool is not open-source.

### Perl

I have learned [Perl, the Practical Extraction and Report
Language](https://strawberryperl.com/), a long time ago and it still helps me
with doing some scripting tasks. So there is no reason for me to switch to
Python or something else.

### Ant

Ant has just been described before and it interacts well with Maven and is
sometimes more simple to use than Maven.

### DevOps

> DevOps is a set of practices that works to automate and integrate the
> processes between software development and IT teams, so they can build, test,
> and release software faster and more reliably.
>
> <cite><a href="https://www.atlassian.com/devops">Atlassian</a></cite>

The tools and techniques I have described can be used from simple to
complex. So from a single person running Maven from the command line to a
team using [Jenkins](https://www.jenkins.io/) and the free artifact repository
[Nexus](https://www.sonatype.com/nexus/repository-oss) to build a [Continuous
Deployment](https://www.atlassian.com/continuous-delivery/principles/continuous-integration-vs-delivery-vs-deployment)
pipeline based on Maven.

# Conclusion

I hope I have given you enough appetite to continue reading this series of
articles about building Oracle applications. Apart from the Oracle Database
almost all tools are open source (and mature) so you can use that argument to
convince your boss. And some tools also have a (paid) support option if that
is needed.

Stay tuned!
