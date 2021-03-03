---
title: About Oracle APEX and translations (2)
categories: development
tags: [ Oracle, APEX, Translation, DevOps ]
permalink: /oracle-apex-translation-2/
toc: true
toc_label: "Table of contents"
toc_icon: "globe"
---

<figure style="width: 300px" class="centered">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/earth-globe-3-1451708-640x640.jpg" alt="">
	<figcaption>Free earth globe Images and Royalty-free Stock Photos, https://www.freeimages.com/</figcaption>
</figure> 

The last Blog I talked about this subject too was [About Oracle APEX and translations]({{ site.url }}{{ site.baseurl }}/oracle-apex-translation/).
The last question was:

> The next problem is how you define your text messages using scripting.

<!--more-->

# Maintain text messages using PL/SQL

Well, the APEX API should give us a clue:

## APEX 18.2 API

> You can use APEX_LANG API to translate messages.
>
> - CREATE_LANGUAGE_MAPPING Procedure
> - DELETE_LANGUAGE_MAPPING Procedure
> - EMIT_LANGUAGE_SELECTOR_LIST Procedure
> - LANG Function
> - MESSAGE Function
> - PUBLISH_APPLICATION Procedure
> - SEED_TRANSLATIONS Procedure
> - UPDATE_LANGUAGE_MAPPING Procedure
> - UPDATE_MESSAGE Procedure
> - UPDATE_TRANSLATED_STRING Procedure

> <cite><a href="https://docs.oracle.com/en/database/oracle/application-express/18.2/aeapi/APEX_LANG.html#GUID-68DF9D22-3C3A-418A-B27A-868A569BD990">APEX 18.2</a></cite>


## APEX_LANG.UPDATE_MESSAGE signature


```
APEX_LANG.UPDATE_MESSAGE (
  p_id IN NUMBER,
  p_message_text IN VARCHAR2 )
```

| Parameter		   | Description                                     |
| ---------      | -----------                                     |
| p_id           | The ID of the text message.                     |
| p_message_text | The new text for the translatable text message. |


## APEX 19.2 API

In APEX 19.2 they updated the documentation and added the CREATE_MESSAGE and DELETE_MESSAGE procedures.
Those procedures were already part of the APEX 18.2 HTMLDB_LANG package (APEX_LANG is just a synonym for HTMLDB_LANG) but not mentioned in its documentation.

> You can use APEX_LANG API to translate messages.
> 
> - CREATE_MESSAGE Procedure
> - DELETE_MESSAGE Procedure
> - CREATE_LANGUAGE_MAPPING Procedure
> - DELETE_LANGUAGE_MAPPING Procedure
> - EMIT_LANGUAGE_SELECTOR_LIST Procedure
> - LANG Function
> - MESSAGE Function
> - PUBLISH_APPLICATION Procedure
> - SEED_TRANSLATIONS Procedure
> - UPDATE_LANGUAGE_MAPPING Procedure
> - UPDATE_MESSAGE Procedure
> - UPDATE_TRANSLATED_STRING Procedure

> <cite><a href="https://docs.oracle.com/en/database/oracle/application-express/19.2/aeapi/APEX_LANG.html#GUID-68DF9D22-3C3A-418A-B27A-868A569BD990">APEX 19.2</a></cite>


## APEX_LANG.CREATE_MESSAGE signature


```
APEX_LANG.CREATE_MESSAGE (
  p_application_id           IN NUMBER,
  p_name                     IN VARCHAR2,
  p_language                 IN VARCHAR2,
  p_message_text             IN VARCHAR2 )
```

| Parameter        | Description                                                                                                                               |
| ---------        | -----------                                                                                                                               | 
| p_application_id | The ID of the application for which you wish to create the translatable text message. This is the ID of the primary language application. |
| p_name 					 | The name of the translatable text message.  		 												 							 				 		 			 		 				 											 |
| p_language 			 | The IANA language code for the mapping. Examples include en-us, fr-ca, ja, he. 																													 |
| p_message 			 | The text of the translatable text message. 										 																																					 |


## APEX_LANG.DELETE_MESSAGE signature

```
APEX_LANG.DELETE_MESSAGE (
  p_id IN NUMBER )
```

| Parameter | Description                 |
| --------- | -----------                 |
| p_id 			| The ID of the text message. |


When you look at the three signatures and the rest of the APEX documentation / packages / dictionary there seems to be no way to get the ID of the text message, as strange as it seems.


## How to get the ID of the text message?

Thanks to the Oracle APEX community I stumbled on this:


> I can also list the current ones through `select * from APEX_APPLICATION_TRANSLATIONS where application_id = <my_app>`

> <cite><a href="https://community.oracle.com/tech/developers/discussion/716972/translating-messages-used-internally-by-apex-create-through-plsql">"Translating Messages used internally by apex" - create through plsql?</a></cite>.



So this code retrieves the ID of a text message identified by its application id, name and language:

```
select  aat.translation_entry_id
into    :p_id
from    apex_application_translations aat
where   aat.application_id = :p_application_id
and     aat.translatable_message = :p_name
and     aat.language_code = :p_language
```

# Conclusion

So using the three signatures and the query to retrieve the ID, you can create
a nice API to select and maintain APEX text messages. And you can use that in
your DevOps process to automatically insert, update, merge or delete
messages. Combined with the `APP_TEXT$Message_Name` syntax mentioned in my
previous post about this subject, you can already translate an important part
of your application without going thru the tedious process Oracle APEX
proposes for translating an application.
