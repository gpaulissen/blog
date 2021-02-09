---
title: About Oracle Apex and translations
categories: development
tags: [ Oracle, Apex, Translation ]
permalink: /oracle-apex-translation/

---

![no-alignment]({{ site.url }}{{ site.baseurl }}/assets/images/earth-globe-3-1451708-640x640.jpg)

In an international environment being able to translate an Apex application without too much effort is a valuable asset.

Traditionally Apex offered a process to translate an application:
- create a shadow language application
- seed the source application
- download a XLIFF file that contains all the text to be translated
- translate it manually or automatically, maybe even by a specialized company since XLIFF is the standard for translation
- upload the translated XLIFF file
- apply the translation
- publish the shadown language application

Quite a process but it works, except that it seems you need to refresh your browser if you run the shadow language application when it has been published.

I have also seen approaches where X number of application items are defined and set by application computations and then those items are used for labels using the substition syntax (e.g. &LBL_LAST_NAME.).

But since Apex 18.2 there is new and not well-known functionality that evolves on this solution without the need to define application items (and computations).

<!--more-->

It is a new form of substitution string, APP_TEXT$Message_Name and APP_TEXT$Message_Name$Lang.

See the [Using Built-in Substitution Strings Apex 18.2](https://docs.oracle.com/en/database/oracle/application-express/18.2/htmdb/understanding-substitution-strings.html#GUID-2FDF06A4-B083-49F8-9061-AE1F5629C659).

So the idea is that you define labels and column headings using this
substitution string construction (e.g. &APP_TEXT$LBL_LAST_NAME.). When you
have defined the text message LBL_LAST_NAME it will display it. The only
problem I have seen is that in validation errors where #LABEL# is used, it
will not substitute the label twice so you may get '&APP_TEXT$LBL_LAST_NAME.
must have some value'. But that is the same when you use application items.

The next problem is how you define your text messages using scripting, but that is a topic for another post...
