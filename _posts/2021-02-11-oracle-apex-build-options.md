---
title: Agile programming using Oracle Apex Build Options
categories: development
tags: [ Oracle, Apex, Agile ]
permalink: /oracle-apex-build-options/
classes: wide
---

Last time I wanted to use a new feature of Apex 19.2, the multi-column Popup
LOV **and** multi return list.  Traditionally the Apex Popup LOV just displays
a description and returns a reference (key). Starting with Apex 19.2 you can
now show more than one column **and** return more than one column.  And what I
wanted was to show a list of contacts and return name, code, telephone number
and some more info. Till 19.2 I had to use dynamic Ajax calls to get each
column, an approach which worked (most of the time). So I wanted to use
this new feature and it seemed to work ... till the page was refreshed and all
returned columns were empty.

The [list of Apex 19.2 known issues](https://www.oracle.com/tools/downloads/apex-downloads/apex-192-known-issues.html) described this bug well:

> 30537256 - POPUP LOV: ADDITIONAL OUTPUT ITEMS GET CLEARED ON PAGE LOAD
> With the new Popup LOV, it is possible to define 'Additional Output' items, which allow multiple LOV values to be returned into external page items or Interactive Grid columns.
> At present though, if you do this to say populate additional items on a form (for example ADDRESS1, ADDRESS2 after selecting a post code / zip code from a popup).
> When you open that record again, even if ADDRESS1 / ADDRESS2 are stored in the DB, they will appear empty.
> This is because the Popup LOV item type incorrectly clears any additional output items out on page load.
> Solution: A PSE has been made available for this issue on MyOracleSupport.
> Navigate to Patches & Updates and search for patch number 30392181.
> Note: this PSE also contains other bug fixes, please review the README in the patch for more details.

So I had to install patch 30392181. In the development environment this was
easy but not in production where I had to wait for the DBA approval. So I
could not continue?  No, I just could continue to add this feature as a Build
Option named after the patch to install. I set the default value for the Build
Option to False (patch not installed) and all you have to do is to set it to
True in production after the patch is installed.

The new application logic based on the Popup LOV can be constrained by the
build option being True. And the old application logic can be constrained by
the build option being False.

Thanks, Apex, for this clever piece of work. So you can continue to develop
nice features by just cleverly using build options and (de-)active them as
necessary. Really Agile!
