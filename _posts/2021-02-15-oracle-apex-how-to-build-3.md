---
title: How to build an Oracle APEX application (3)
categories: development
tags: [ Oracle, APEX, Virtualbox, VirtualMachine ]
permalink: /oracle-apex-how-to-build-3/
toc: true
toc_label: "Table of contents"
toc_icon: "database"
excerpt: "This time I will elaborate on the base tools, the Oracle Database and Oracle APEX."
last_modified_at: 2021-03-04T12:26:00
---

<figure class="centered">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/512px-Devops-toolchain.svg.png" alt="">
	<figcaption>Kharnagy, CC BY-SA 4.0 <https://creativecommons.org/licenses/by-sa/4.0>, via Wikimedia Commons</figcaption>
</figure> 

Last time in ["How to build an Oracle APEX application (2)"]({{ site.url }}{{
site.baseurl }}/oracle-apex-how-to-build-2/), I did show you the database structure.

This time I will elaborate on the base tools, the Oracle Database and Oracle APEX.

# Oracle Database

## How to use it?

I can tell a lot about the database but I will focus on the essentials needed to
work with a front-end like APEX.

> I recently saw this approach used in a complex APEX application built for my current client, and I liked what I saw - so I used a similar one in another project of mine, with good results.
>
> 1. Pages load and process faster
> 2. Less PL/SQL compilation at runtime
> 3. Code is more maintainable and reusable
> 4. Database object dependency analysis is much more reliable
> 5. APEX application export files are smaller - faster to deploy
> 6. APEX pages can be copied and adapted (e.g. for different interfaces) easier
>
> <p><img loading="lazy" class="alignnone size-full wp-image-1709" alt="ratsnest-app" src="https://jeffkemponoracle.com/wp-content/uploads/2014/02/ratsnest-app.jpg" width="450" height="191" srcset="https://jeffkemponoracle.com/wp-content/uploads/2014/02/ratsnest-app.jpg 450w, https://jeffkemponoracle.com/wp-content/uploads/2014/02/ratsnest-app-300x127.jpg 300w" sizes="(max-width: 450px) 100vw, 450px" /><br />
>
> <cite><a href="https://jeffkemponoracle.com/tag/best-practice/">Build your APEX application better - do less in APEX, Jeff Kemp on Oracle, 13 February 2014</a></cite>

I couldn't agree more. This was quite some time ago but it is still valid. For
any front-end actually. This is not just for APEX. I will repeat this again
and again: you have a really powerful database for which you (or your
company/client) have paid a lot. So use it well, put all the data and business
logic where it belongs: the **database**. It will make your application
faster, more secure, easier to develop and debug. And also more maintainable
since there is a lot more Oracle Database expertise than for instance some
obscure front-end expertise. This is a best practice since long, just do it.

And for those people who like database independence: I just like to get things
done **well**, so I use PL/SQL since that is just the best language to work
with the Oracle Database. I do not want to use JDBC (in combination with Java)
or ODBC (in combination with .NET/Python) when I can use PL/SQL. And every
other well designed database has some kind of language like PL/SQL. So if you
want to be independent why not write a language layer for each database having
the same functionality.

One more last thing. I have seen a lot of projects with Java developers using
JDBC and Oracle and what has surprised me very often is the ignorance of the
database they work with. The Java code just issues statements against the
tables, no invocation of PL/SQL (package) procedures or functions. All
functionality in the middle tier, not in the database. And the funny thing is
that Oracle even has an Object Oriented layer: Oracle Object Types. It is true
that Object Types are more limited than Java classes but I have created some
nice applications based on the Oracle OO concept. You can also use an Object
Type as a kind of glue between Oracle and another language like Java. And as a
Java programmer you can also invoke REST APIs powered by PL/SQL. What a pity
that a large part of the Oracle functionality is not used by those Java
programmers.

## What version?

My advice is to use the latest major version or the one before. So for an
[Oracle Database nowadays](https://en.wikipedia.org/wiki/Oracle_Database) this
means version 21c or the long-term support (LTS) release 19c (equivalent with
12.2.0.3). This is a simple advice for any software and it assures you that
you keep up with enhancements, (security) patches and so on. Do not wait to
long. Again I will use the analogy with a house: you'd better paint and
maintain it regularly than wait till the wood has rotten.

## What platform?

I have talked about it briefly in the [first post]({{ site.url }}{{
site.baseurl }}/oracle-apex-how-to-build-1/) but for a development environment
I would just download the prebuilt virtual machine [Database App Development
VM](https://www.oracle.com/downloads/developer-vm/community-downloads.html)
from Oracle for my development environment. It comes with the database and
APEX integrated on an Unbreakable Linux Operation System. Simple to use and
backup and you can be the DBA without bothering anyone. And free of charge for
a development environment.

![Database App Development VM]({{ site.url }}{{ site.baseurl }}/assets/images/database-vm.png)

Do not forget to make snapshots (backups) regularly. It has saved my life
quite a few times.
{: .notice--warning}

## Virtual machine settings

You may have more than one virtual machine (VM) and thus more than one database and
APEX instance and you would like to have them all running and accessible at
the same time? You will need port forwarding to accomplish this.

Please note that the virtual machine network configuration for the database is
the same: ip address 127.0.0.1, port 1521 and instance name ORCL. APEX can be
accessed through port 8080 on the virtual machine.
{: .notice--info}

I have a Windows 10 laptop with two virtual machines, DEV (APEX 18.2) and VM19
(APEX 19.2).

These are the port forwarding rules for VM DEV: ![port forwarding rules for VM DEV]({{ site.url }}{{
site.baseurl }}/assets/images/port-forwarding-rules-dev.png)

And these are the port forwarding rules for VM VM19: ![port forwarding rules for VM VM19]({{ site.url }}{{
site.baseurl }}/assets/images/port-forwarding-rules-vm19.png)

So the DEV database can be accessed by port 1526 on my Windows laptop and the
DEV APEX instance thru the standard port 8080. The VM19 database can be
accessed by port 1527 on my Windows laptop and the DEV APEX instance thru port
8082.

And this is the SQL*Net TNSNAMES configuration: ![SQL*Net TNSNAMES configuration]({{ site.url }}{{
site.baseurl }}/assets/images/tnsnames.png)

I always use the environment variable TNS_ADMIN on any platform to point to the
directory of the SQL\*Net tnsnames.ora file. This allows me to have **one**
point of truth for SQL\*Net even if I have several Oracle product installations.
{: .notice--info}

# Oracle APEX

As said before this series of articles is not an advice how to use the tools but
it is just how to build a database application with a plan, an
architecture. This book, [Oracle APEX Best
Practices](https://www.packtpub.com/product/oracle-apex-best-practices/9781849684002),
may help you with that.

Anyhow if you have followed my advice to use a virtual machine for
development, you have an APEX instance now.

For me the way to go from development to production is to export the
development application and import it in every next stage till it reaches
production. I do not consider it a very good idea to manually apply the
changes in later stages. It is certainly not the DevOps way.

## Collaborating

If you need to collaborate while developing an application you need of course
a shared database and APEX instance. It will be a little bit more difficult
since you need to be more careful but thanks to the ability to lock APEX pages
and the Build Options you can manage.

## Parallel development

The problem with APEX is that you can not really install parts of it: it is
all or nothing. So even if you split the APEX application export file using
the [Oracle SQLcl client](https://www.oracle.com/database/technologies/appdev/sqlcl.html),
you can not just use some files. You have to use them all.

This influences also parallel development (branching if you prefer).

> My current client has a large number of APEX applications, one of which is a
> doozy. It is a mission-critical and complex application in APEX 4.0.2 used
> throughout the business, with an impressively long list of features, with an
> equally impressively long list of enhancement requests in the queue.
>
> They always have a number of projects on the go with it, and they wanted us to
> develop two major revisions to it in parallel. In other words, we'd have v1.0
> (so to speak) in Production, which still needed support and urgent defect
> fixing, v1.1 in Dev1 for project A, and v1.2 in Dev2 for project B. Oh, and we
> don't know if Project A will go live before Project B, or vice versa. We have
> source control, so we should be able to branch the application and have
> separate teams working on each branch, right?
>
> We said, "no way". Trying to merge changes from a branch of an APEX app into
> an existing APEX app is not going to work, practically speaking. The merged
> script would most likely fail to run at all, or if it somehow magically runs,
> it'd probably break something.
>
> <cite><a
  href="https://jeffkemponoracle.com/2014/01/parallel-development-in-apex/">Parallel
  Development in APEX, Jeff Kemp on Oracle, 23 January 2014</a></cite>

Things have not really changed since 2014. 

## Keep APEX versions aligned

Keep in mind that you cannot import an APEX application into another APEX
instance if the exported version is **higher** than the version of APEX to
import into. So exporting an APEX 19.2 application will **not** import into
APEX 18.2. So align all your APEX versions from development till production.
{: .notice--warning}

# Conclusion

In this post you have seen how to setup an Oracle APEX development environment
and some best practices as well.

Stay tuned!
