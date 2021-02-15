---
title: How to build an Oracle Apex application (3)
categories: development
tags: [ Oracle, Apex, DevOps, DataModeling, Virtualbox, VirtualMachine ]
permalink: /oracle-apex-how-to-build-3/
toc: true
toc_label: "Table of contents"
toc_icon: "database"
---

<figure class="centered">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/512px-Devops-toolchain.svg.png" alt="">
	<figcaption>Kharnagy, CC BY-SA 4.0 <https://creativecommons.org/licenses/by-sa/4.0>, via Wikimedia Commons</figcaption>
</figure> 

Last time in ["How to build an Oracle Apex application (2)"]({{ site.url }}{{
site.baseurl }}/oracle-apex-how-to-build-2/), I did show you the database structure.

This time I will elaborate on the base tools, the Oracle database and Oracle Apex.

# Oracle database

## How to use it?

I can tell a lot about the database but I will focus on the essentials needed to
work with a front-end like Apex.

> I recently saw this approach used in a complex Apex application built for my current client, and I liked what I saw - so I used a similar one in another project of mine, with good results.
>
> 1. Pages load and process faster
> 2. Less PL/SQL compilation at runtime
> 3. Code is more maintainable and reusable
> 4. Database object dependency analysis is much more reliable
> 5. Apex application export files are smaller - faster to deploy
> 6. Apex pages can be copied and adapted (e.g. for different interfaces) easier
>
> <p><img loading="lazy" class="alignnone size-full wp-image-1709" alt="ratsnest-app" src="https://jeffkemponoracle.com/wp-content/uploads/2014/02/ratsnest-app.jpg" width="450" height="191" srcset="https://jeffkemponoracle.com/wp-content/uploads/2014/02/ratsnest-app.jpg 450w, https://jeffkemponoracle.com/wp-content/uploads/2014/02/ratsnest-app-300x127.jpg 300w" sizes="(max-width: 450px) 100vw, 450px" /><br />
>
> <cite><a href="https://jeffkemponoracle.com/tag/best-practice/">Build your APEX application better - do less in APEX, Jeff Kemp on Oracle, 13 February 2014</a></cite>

I couldn't agree more. This was quite some time ago but it is still valid. For
any front-end actually. This is not just for Apex. I will repeat this again
and again: you have a really powerful database for which you (or your
company/client) have paid a lot. So use it well, put all the data and business
logic where it belongs: the **database**. It will make your application
faster, more secure, easier to develop and debug. And also more maintainable
since there is a lot more Oracle database expertise than for instance some
obscure front-end expertise. This is a best practice since long, just do it.

And for those people who like database independence: I just like to get things
done **well**, so I use PL/SQL since that is just the best language to work
with the Oracle database. I do not want to use JDBC/ODBC when I can use
PL/SQL. And every other well designed database has some kind of language like
PL/SQL so if you want to be independent why not write a language layer for
each database having the same functionality.

One more last thing. I have seen a lot of projects with Java developers
using Jdbc and Oracle and what has surprised me very often is the ignorance of
the database they work with. The Java code just issues statements against the
tables, no invocation of PL/SQL (package) procedures or functions. All
functionality in the middle tier, not in the database. And the funny thing is
that Oracle even has an Object Oriented layer: Oracle Object Types. What a pity
that a large part of the Oracle functionality is not used by those Java programmers.

## What version?

My advice is to use the latest major version or the one before. So for an [Oracle
Database nowadays](https://en.wikipedia.org/wiki/Oracle_Database) this means version
21c or 19c (equivalent with 12.2.0.3). This is a
simple advice for any software and it assures you that you keep up with
enhancements, (security) patches and so on. Do not wait too long. Again I
will use the analogy with a house: you'd better paint and maintain it
regularly than wait till the wood has rotten.

## What platform?

I have talked about it briefly in the [first post]({{ site.url }}{{
site.baseurl }}/oracle-apex-how-to-build-2/) but for a development environment
I would just download a virtual development machine. It comes with the
database and Apex integrated on an Unbreakable Linux Operation System. Simple
to use and backup and you can be a DBA without bothering anyone. And free of
charge for a development environment.

Do not forget to make snapshots (backups) regularly. It has saved my life
quite a few times.
{: .notice--warning}

## Virtual machine settings

You may have more than one virtual machine (VM) and thus more than one database and
Apex instance and you would like to have them all running and accessible at
the same time? You will need port forwarding to accomplish this.

Please note that the virtual machine network configuration for the database is
the same: ip address 127.0.0.1, port 1521 and instance name ORCL. Apex can be
accessed through port 8080 on the virtual machine.
{: .notice--info}

I have a Windows 10 laptop with two virtual machines, DEV (Apex 18.2) and VM19
(Apex 19.2).

These are the port forwarding rules for VM DEV: ![port forwarding rules for VM DEV]({{ site.url }}{{
site.baseurl }}/assets/images/port-forwarding-rules-dev.png)

And these are the port forwarding rules for VM VM19: ![port forwarding rules for VM VM19]({{ site.url }}{{
site.baseurl }}/assets/images/port-forwarding-rules-vm19.png)

So the DEV database can be accessed by port 1526 on my Windows laptop and the
DEV Apex instance thru the standard port 8080. The VM19 database can be
accessed by port 1527 on my Windows laptop and the DEV Apex instance thru port
8082.

And this is the SQL*Net TNSNAMES configuration: ![SQL*Net TNSNAMES configuration]({{ site.url }}{{
site.baseurl }}/assets/images/tnsnames.png)

I always use the environment variable TNS_ADMIN on any platform to point to the
directory of the SQL\*Net tnsnames.ora file. This allows me to have **one**
point of truth for SQL\*Net even if I have several Oracle product installations.
{: .notice--info}

# Oracle Apex

As said before this series of articles is not an advice how to use the tools but
it is just how to build a database application with a plan, an
architecture. This book, [Oracle APEX Best
Practices](https://www.packtpub.com/product/oracle-apex-best-practices/9781849684002),
may help you with that.

Anyhow if you have followed my advice to use a virtual machine for
development, you have an Apex instance now.

If you need to collaborate while developing an application you need of course
a shared database and Apex instance. It will be a little bit more difficult
since you need to be more careful but thanks to the ability to lock Apex pages
and to define Build Options you can manage.

For me the way to go from development to production is to export the
development application and import it in every next stage till it reaches
production. I do not consider it a very good idea to manually apply the
changes in later stages. It is certainly not the DevOps way.

Keep in mind that you cannot import an Apex application into another Apex
instance if the exported version is **higher** than the version of Apex to
import into. So exporting an Apex 19.2 application will **not** import into
Apex 18.2. So align all your Apex versions from development till production.
{: .notice--warning}

# Conclusion

In this post you have seen how to setup an Oracle APEX development environment
and some best practices as well.

Stay tuned!
