---
title: About ING and dumping data to files
categories: development
tags: [ Oracle, PRO*C, C ]
permalink: /oradumper/
---
Yesterday I got an email from LinkedIn: do you have plans to move back to the Netherlands?

<!--more-->

The mail was from someone I knew quite well. In 2006 I started to work for ING
Netherlands in Amsterdam, actually the old ING Barings IT department. He was
the contact person from a subcontractor (big companies almost always have some
preferred IT subcontractor who hire people as needed). It turned out to be a
long and interesting period. I was there when Lehmann Brothers fell down.

Actually the question was whether I could help with upgrading an
application I had designed back then with another very smart and nice
colleague and under the supervision of a supportive and intelligent
manager. Thanks again, Kenneth and Glenn (and a lot of others since it was a
very nice working environment).

The application I had designed gathered financial information from all over
the world (via files, database links or whatever) and that should be able to
reproduce it when needed, even though the structure version had changed in the
meantime (like fields added or removed).

When I started there there was already a first release based on an Oracle ETL
product. But it was not the ideal product because it had no functionality for
the versioning aspect and performance was an issue as well (all data received
was stored in tables and that could take a lot of time).

The idea I came up with was simple:
- try hard to be lazy (do nothing unless needed)
- store the data gathered as files
- use Oracle external tables to read those stored files
- use Oracle Objects to cater for all the possibilities of in- and outgoing data
- use versioning in the meta data

Performance was immediately no more an issue at all. For example when the
financial information was transferred as a file, just that file was just
stored on the server file system. No DML needed. And the versioning aspect was
taken care of as well. The manager approved of this idea and kept us out of
the loop while my colleague and I started on the next release of this product.

It was an interesting project because we would use Object Oriented
Programming, something that was common in the Java community at that time but
certainly not in the Oracle community although Oracle Objects existed already
quite some time.

The only problem left was how to convert financial information that came from other
database via a link into a file? And as quick as possible!

So I invented oradumper, an Oracle utility that takes as input a SQL query and
outputs a file, either fixed or variable format and with or without
headers. The idea was based on a PRO*C program by 
[Tom Kyte](https://asktom.oracle.com/pls/asktom/f?p=100:11:0::::P11_QUESTION_ID:459020243348).

The utility I have written and maintained since then is UTF aware and written
in ANSI embedded SQL and one of my favorite languages, C. All relevant
datatypes except CLOBs and BLOBs are treated correctly. But hey, how do you want
to store a CLOB column as a fixed size column for more than one row?

Anyway, the latest source can be found at
[Sourceforge](https://sourceforge.net/p/transferware/code/HEAD/tree/trunk/oradumper/),
the open source site that was in vogue when I started to open source the
software of the company I worked for in 2002: [Transfer
Solutions](https://www.transfer-solutions.com/).

And, for the time being, I have no plans to move back to the Netherlands. Sorry, Pieter.
