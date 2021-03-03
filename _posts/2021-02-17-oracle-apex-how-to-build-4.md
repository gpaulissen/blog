---
title: How to build an Oracle APEX application (4)
categories: development
tags: [ Oracle, APEX, DataModeling ]
permalink: /oracle-apex-how-to-build-4/
toc: true
toc_label: "Table of contents"
toc_icon: "database"
excerpt: "This time: Oracle SQL Developer Data Modeler."
classes: wide
last_modified_at: 2021-03-03T15:20:00
---

<figure class="centered">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/512px-Devops-toolchain.svg.png" alt="">
	<figcaption>Kharnagy, CC BY-SA 4.0 <https://creativecommons.org/licenses/by-sa/4.0>, via Wikimedia Commons</figcaption>
</figure>

Last time in ["How to build an Oracle APEX application (3)"]({{ site.url }}{{
site.baseurl }}/oracle-apex-how-to-build-3/), I told you about the Oracle Database and Oracle APEX.

This time I will discuss Oracle SQL Developer Data Modeler.

# Oracle SQL Developer Data Modeler

A book I can recommend is [Oracle SQL Developer Data Modeler for Database Design Mastery](https://www.goodreads.com/book/show/23871562-oracle-sql-developer-data-modeler-for-database-design-mastery) by [Heli Helskyaho](https://helifromfinland.blog/).

## Just modeling

I use Data Modeler mainly for modeling, documentation and generating DDL
scripts for the initial setup and incremental migration scripts later on. For
other activities I use tools that suit me better, the Unix approach.

This utility allows you to define views but I do **not** use it since it gave me
a lot of problems. A simple SQL script to create the view is just enough.
{: .notice--warning}

### Logical Model

This is the Entity Relationship Model area.

You should really take your time to design your model and to verify it using
the Design Rules described later on. This is the foundation of your
application.

And do not forget to use domains whenever appropriate. You can even have one
corporate domains XML file if you prefer.
{: .notice--info}

### Relational Models

Each Logical Model may be transformed into a Relation Model, one for Oracle
Database 12c, Oracle Database 12cR2 and so on. This allows you to use the features of those versions.

My preference is to just one relational model per Logical Model to keep it simple.

Again you should really take your time to design your relational model and to
verify it using the Design Rules described later on. This is the foundation of
your application.

#### Business rules

I am old enough to remember the Business Rules Classification:

<figure class="centered" width="1200px">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/business-rules-classification.png" alt="">
	<figcaption>Business Rules Classification</figcaption>
</figure>

Quite a few business rules can be defined easily using Data Modeler, here some examples:

| business rule                                           | how |
| -------------                                           | --- |
| Department code must be numeric.                        | Column datatype |
| Employee job must be 'CLERK', 'SALES REP' or 'MANAGER'. | Use a domain |
| Employee salary must be a multiple of 1000.             | Domain (which lets you define a constraint) |
| Employee exit date must be later than hire date.        | Table Level Constraints | 

Other constraints may not fit into Data Modeler and may need to be implemented
in another way. For more inspiration I will refer to [implementing business
rules](http://rwijk.blogspot.nl/2008/07/implementing-business-rules.html) by
Rob van Wijk.

I have had difficulties with constraints implemented by materialized views
with refresh fast on commit in an APEX environment. Maybe I did it wrong, maybe the database version
(Oracle Database 12) was a little buggy or maybe it works just nice in theory. I resorted to
triggers and PL/SQL.
{: .notice--warning}


#### Incremental migration scripts

You can define a connection via:

<figure class="centered">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/import-data-dictionary.png" alt="">
	<figcaption>Import Data Dictionary menu option</figcaption>
</figure> 

Then you can use that connection to execute the `Synchronize Data Dictionary`
functionality. This will create an incremental migration script you can use
with Flyway. Sometimes you may need to tweak the generated script.

## Design Rules and Transformations

One of the features I can really recommend are the `Design Rules And Transformations`: ![Design Rules And Transformations menu]({{ site.url }}{{ site.baseurl
}}/assets/images/design-rules-and-transformations.png)

### Design Rules

This is the [`LINT`](https://en.wikipedia.org/wiki/Lint_(software)) like tool
of Data Modeler, an analysis tool that flags errors,
[bugs](https://en.wikipedia.org/wiki/Software_bug), stylistic errors and
suspicious constructs. Applicable for both the Logical Model and Relational
Models.

### Custom Transformation Scripts

This allows you to use predefined scripts to do transformations **and** to
define your own.

Here an example for setting the table name to plural. You usually define the
entity name in singular and the table name in plural. This custom utility (`Table
Names Plural - custom`) allows you to do it automatically:

```
var tables = model.getTableSet().toArray();
for (var t = 0; t<tables.length; t++){
	var table = tables[t];
	var tableName = table.getName();
 	if (tableName.endsWith("Y")) {
 		// Y -> IES
 		table.setName(tableName.slice(0, -1) + "IES");
 		table.setDirty(true);
 	} else if (!tableName.endsWith("S")) {
 		// . -> .S
 		table.setName(tableName + "S");
 		table.setDirty(true);
 	}
}
```

## Configuration

You can better use **one** modeling project for all your applications when you
use SQL Data Modeler so you can share your configuration more easily between
projects and developers.
{: .notice--warning}

From my [GitHub datamodeler project](https://github.com/gpaulissen/datamodeler) here the README:

> A project to share Oracle SQL Datamodeler settings and scripts. Oracle SQL Developer Data Modeler has several global configuration items like:
>
> - preferences
> - design rules and transformations
> - default domains
>
> Besides that there are also design preferences and glossaries but you can store them in a version control system easily unlike the global configuration.
>
> The official way to share the global configuration between computers is to use the various import and export utilities from the Data Modeler. However this is quite time consuming and thus error prone.
> 
> An easier approach is to just backup these settings to a directory you specify
> as a command line option (ideally under version control). Then you can restore
> them when needed. This project tries to accomplish just that: KISS.
>
> <cite><a href="https://github.com/gpaulissen/datamodeler/blob/master/README.md">Oracle SQL Developer Data Modeler configuration</a></cite>

It is just a simpler and more friendly approach than using manual export and import
actions between developers.

If you collaborate with others you had better keep all the folders and files
the same since the configuration contains those names.
{: .notice--warning}

# Conclusion

Here I shared some ideas about using SQL Developer Data Modeler, a tool that
can construct the foundation of your application very well.

Stay tuned!
