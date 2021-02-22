---
title: How to build an Oracle Apex application (5)
categories: development
tags: [ Oracle, Apex, Git, Subversion, Maven, Flyway ]
permalink: /oracle-apex-how-to-build-5/
toc: true
toc_label: "Table of contents"
toc_icon: "database"
excerpt: "This time: Git, Subversion, Maven, Flyway."
---

<figure class="centered">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/512px-Devops-toolchain.svg.png" alt="">
	<figcaption>Kharnagy, CC BY-SA 4.0 <https://creativecommons.org/licenses/by-sa/4.0>, via Wikimedia Commons</figcaption>
</figure>

Last time in ["How to build an Oracle Apex application (4)"]({{ site.url }}{{
site.baseurl }}/oracle-apex-how-to-build-4/), I told you about the Oracle SQL
Developer Data Modeler.

This time I will discuss the following tools: Git, Subversion, Maven and Flyway.

# Flyway

The first tool I would like to discuss is one of the cornerstones of the build
architecture.

## Why migrations?

For the non-database code side of projects we are now in control.

From the Flyway documentation:

> - Version control is now universal with better tools everyday.
> - We have reproducible builds and continuous integration.
> - We have well defined release and deployment processes.
> 
> **But what about the database?**
> Well unfortunately we have not been doing so well there.
> Many projects still rely on manually applied sql scripts.
> And sometimes not even that (a quick sql statement here or there to fix a problem).
> And soon many questions arise:
> - What state is the database in on this machine?
> - Has this script already been applied or not?
> - Has the quick fix in production been applied in test afterwards?
> - How do you set up a new database instance?
> - More often than not the answer to these questions is: We don't know.
>
> **Database migrations are a great way to regain control of this mess.**
>
> They allow you to:
> - Recreate a database from scratch
> - Make it clear at all times what state a database is in
> - Migrate in a deterministic way from your current version of the database to a newer one
>
> <cite><a href="https://flywaydb.org/documentation/getstarted/why">Why database migrations?</a></cite>

## How Flyway works?

Again from the Flyway documentation:

1. Flyway uses a schema history table (automatically created by Flyway) to
maintain the track the state of the database.
2. Flyway will scan the filesystem or the classpath of the application for
migrations. They can be written in either Sql or Java.
3. The migrations are then sorted based on their version number and applied in
order.
4. As each migration gets applied, the schema history table is updated
accordingly.
5. With the metadata and the initial state in place, we can now talk about
migrating to newer versions.
6. Flyway will once again scan the filesystem or the classpath of the
application for migrations. The migrations are checked against the schema
history table. If their version number is lower or equal to the one of the
version marked as current, they are ignored.
7. The remaining migrations are the pending migrations: available, but not
applied.

And that's it! Every time the need to evolve the database arises, whether
structure (DDL) or reference data (DML), simply create a new migration with a
version number higher than the current one. The next time Flyway starts, it
will find it and upgrade the database accordingly.

### Incremental migrations

As the name already indicates these files are run only once in a
database. They are usually used for SQL commands that can execute only once:
- CREATE ...
- ALTER ...
- DROP ...

The default naming convention of Flyway is that incremental migrations have a
prefix V, a version number, two underscores and the rest is free.

I prefer to have a timestamp as version number in the Oracle date format
YYYYMMDDHH24MISS. An example is thus V20210217140700__create_table_TEST.sql.

You can put more than one SQL command in an incremental migration but take
care: if the script fails after having executed successfully at least one
command you are in deep trouble. That's why I prefer to have just a single
command in each incremental script **or** I write them foolproof, guarding
against unexpected situations. It depends.

### Repeatable migrations

Repeatable migrations are very useful for managing database objects whose
definition can then simply be maintained in a single file in version
control. Instead of being run just once, they are (re-)applied every time
their checksum changes.

They are typically used for:
- (Re-)creating views/procedures/functions/packages/
- Bulk reference data reinserts

With Flyway's default naming convention, the filename will be similar to the
regular migrations, except for the V prefix which is now replaced with a R and
the lack of a version.

Although there is no order due to a version there is an order because of the
name of the file. In order to minimize the number of errors or warnings during
a migration I use the following naming convention:

```
R__<type order number>.<schema>.<type>.<name>.sql
```

This table shows the types (from DBMS_METADATA) and their type order number:

| type         | type order number |
| ----         | ----------------- |
| FUNCTION     | 08 |
| PACKAGE_SPEC | 09 |
| VIEW 				 | 10 |
| PROCEDURE 	 | 11 |
| PACKAGE_BODY | 14 |
| TYPE_BODY 	 | 15 |
| TRIGGER 		 | 17 |
| OBJECT_GRANT | 18 |
| SYNONYM 		 | 21 |
| COMMENT 		 | 22 |
| JAVA_SOURCE  | 25 |

The missing numbers are used for CREATE only objects like tables, constraints
and so on but they are not used for repeatable migrations so I left them out.

When the schema is not fixed I do not use the \<schema\> part as in
R__09.PACKAGE_SPEC.CFG_PKG.sql, a package specification that defines some
constants (debugging on/off and testing on/off) to be used in conditional compiling.
{: .notice--info}

You must be careful with views. If you create a view that depends on another
object (a view for instance) that has not yet been created, Flyway will fail
**unless** you add the `FORCE` keyword.
{: .notice--warning}

You must be careful with views and instead of triggers. Instead triggers have
the nasty characteristic of disappearing when you recreate the view. But there is a
simple solution. Create the instead of trigger in the **same** script as the
view (creating the view first obviously). Then Flyway will be your savior.
{: .notice--warning}

## DML

As already said DML scripts can be either incremental or repeatable
migrations.

## Preferred order of migrations

The preferred order is:
- incremental migrations
- repeatable DDL migrations
- repeatable DML migrations

You can influence that by choosing wisely the Flyway locations to search for
migration scripts.

## Why not Liquibase?

There is another competitor of Flyway: Liquibase.

I have investigated Liquibase long time ago and I saw recently that the
Oracle SQLcl client supports Liquibase. I still prefer Flyway because it is so
much easier to understand and use. And it handles PL/SQL code so much better.

I will quote this from an [oracle-base.com article](https://oracle-base.com/articles/misc/sqlcl-automating-your-database-deployments-using-sqlcl-and-liquibase):

> **That's Not How You Use It!**
> When you look at examples of using Liquibase on the internet they all have a few things in common.
>
> - They are typically examples used to track table changes and not much else.
> - Like my examples, they are based on small simple schemas. This always makes sense, but issues arise with some methods when things grow.
> - They don't include code objects (procedure, functions, packages, triggers, types etc.).
> - If they do include code objects, they assume each version of the code is in a new file. This means you're going to lose the conventional commit history of a file you would normally expect for code. Instead you have to manually diff between separate files.
> - They assume people need to rollback changes to previous versions of the database through this mechanism. I think creating a rollback script for each schema change makes sense, but I think it's a bad idea to include it in this mechanism. In my opinion all changes should move forward. So a "rollback" is really a new change applied to the database that reverts the changes. This is especially true of code related functionality.
>
> The major issue for me is the way code objects are managed. This may not affect you if you never have code in the database, but for a PL/SQL developer, this feels like a show-stopper. As a result, I prefer to work using scripts, which are kept in source control, and use Liquibase as the deployment and sequencing mechanism. I'm sure many Liquibase users will not like this, and will think I'm using it incorrectly. That's fine. There's more discussion about script management here.

I can only add: if you prefer to have your PL/SQL code in a script why not
your tables and so on (the incremental scripts)?

I rest my case.

# Maven

The reason I have chosen Maven to be the build integration tool is its
excellent support for Flyway and Jenkins. It enables you to do `Continuous
Integration`. And yes, it is used mainly by `Java` projects but who cares!

There is a lot of documentation about Maven but for building Oracle projects
you can just begin with the:
- [POM reference](https://maven.apache.org/pom.html)
- [Flyway Maven Plugin](https://flywaydb.org/documentation/usage/maven/)

# Version control

As already stated before version control tools are necessary for a mature
project. 

## Tools

The tools used nowadays are Git and Subversion.

### Git

Git is nowadays the standard version control tool and also the standard for
GitHub.com, the standard Open Source site. Not a real choice thus.

### Subversion

Subversion is used because Oracle SQL Developer Data Modeler only supports
this version control tool. However, luckily there is a Git Subversion bridge
that allows you to treat a Git repository as a Subversion repository.

Another advantage of Subversion is that it allows you to use the [Maven SCM
plugin](https://maven.apache.org/scm/maven-scm-plugin/) with the `scm:update`
command. This comes in handy when you are on a Citrix server and have no way
to use the command line to clone (checkout) the repository for instance. Then it is
**very** useful to `scm:checkout` your repository once and later update it. For one
reason or another this `scm:update` comand does not seem to work with Git.

So that's why I add this code in the project parent POM:

```
<scm>
  <developerConnection>scm:svn:https://github.com/<user>/<project>.git/trunk</developerConnection>
</scm>
```

See for more information: [Support for Subversion clients,
Github.com](https://docs.github.com/en/github/importing-your-projects-to-github/support-for-subversion-clients)

## Branching or not?

I am not a big fan of branching, especially not in a database environment. I
prefer to have a development process where you implement features. The code
changes initially do not impact production code by using constructs like
conditional compiling (available since Oracle 10) and Apex Build Options (or
if nothing else is available if/then/else) based on a configuration (for
instance a package header defining some boolean constants). Those
constructions allow you to enable/disable parts of the code. In the database
you could even go further using Edition Based Redefinition (EBR) but that
seems only necessary for applications running 24x7.

# Conclusion

In this artcile I have shown you the reasons for choosing Flyway, Maven, Git and
Subversion. The integration between them is (very) good and thus it allows you to do
`Continuous Integration`.

Stay tuned!
