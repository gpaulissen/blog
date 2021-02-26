---
title: How to build an Oracle Apex application (6)
categories: development
tags: [ Oracle, Apex, DevOps, SqlDeveloper, utPLSQL, Perl, Ant, DevOps ]
permalink: /oracle-apex-how-to-build-6/
toc: true
toc_label: "Table of contents"
toc_icon: "database"
excerpt: "This last article: Oracle SQL Developer, utPLSQL, Perl, Ant and DevOps."
---

<figure class="centered">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/512px-Devops-toolchain.svg.png" alt="">
	<figcaption>Kharnagy, CC BY-SA 4.0 <https://creativecommons.org/licenses/by-sa/4.0>, via Wikimedia Commons</figcaption>
</figure>

Last time in ["How to build an Oracle Apex application (5)"]({{ site.url }}{{
site.baseurl }}/oracle-apex-how-to-build-5/), I told you about Git, Subversion, Maven and Flyway.

In this final article, I will discuss the following tools & methods: Oracle SQL Developer,
utPLSQL, Perl, Ant and DevOps.

# Oracle SQL Developer

I will only discuss the External Tools section that I have used on a Citrix
server since it was the only way to launch Maven.

In the examples I assume that an environment variable MAVEN_HOME is defined that points
to the Maven home environment. 

## External tools

Go to the `Tools` menu and click `External Tools`.

### Maven scm:update install

This command is used to update a project from a source repository and install
the package contents in the local Maven repository. I have an Oracle build project that is used
by all other Maven projects and it needs to be installed (locally) before it
can be referenced as a dependency by those other projects.

To create this external tool use executable `java` and arguments:

```
-classpath "${env:var=MAVEN_HOME}\boot\plexus-classworlds-2.6.0.jar" "-Dclassworlds.conf=${env:var=MAVEN_HOME}\bin\m2.conf" "-Dmaven.home=${env:var=MAVEN_HOME}" "-Dlibrary.jansi.path=${env:var=MAVEN_HOME}\lib\jansi-native" "-Dmaven.multiModuleProjectDirectory=${file.dir}" org.codehaus.plexus.classworlds.launcher.Launcher clean scm:update scm:status scm:check-local-modification install
```

The Run directory is:

```
${file.dir}
```

This shows it all: ![Maven scm:update install]({{ site.url }}{{ site.baseurl
}}/assets/images/external-tools-2.png)

The next window allows you to define a meaningful name in the Display window: ![Display]({{ site.url }}{{ site.baseurl
}}/assets/images/external-tools-3.png)

And finally you have to use these settings in Integration: ![Integration]({{ site.url }}{{ site.baseurl
}}/assets/images/external-tools-4.png)

This will allow you to invoke a tool (Maven command) from a POM file using the Right Click
menu:![Right Click menu]({{ site.url }}{{ site.baseurl }}/assets/images/external-tools-5.png)


### Maven db

This invocation of Maven is 1 level lower than the project root as you can see
from `-Dmaven.multiModuleProjectDirectory`:

```
-classpath "${env:var=MAVEN_HOME}\boot\plexus-classworlds-2.6.0.jar" "-Dclassworlds.conf=${env:var=MAVEN_HOME}\bin\m2.conf" "-Dmaven.home=${env:var=MAVEN_HOME}" "-Dlibrary.jansi.path=${env:var=MAVEN_HOME}\lib\jansi-native" "-Dmaven.multiModuleProjectDirectory=${file.dir}/.." org.codehaus.plexus.classworlds.launcher.Launcher compile flyway:repair flyway:migrate ${promptl:=option1} ${promptl:=option2}
```

### Maven apex

Same level as db, other goals:

```
-classpath "${env:var=MAVEN_HOME}\boot\plexus-classworlds-2.6.0.jar" "-Dclassworlds.conf=${env:var=MAVEN_HOME}\bin\m2.conf" "-Dmaven.home=${env:var=MAVEN_HOME}" "-Dlibrary.jansi.path=${env:var=MAVEN_HOME}\lib\jansi-native" "-Dmaven.multiModuleProjectDirectory=${file.dir}/.." org.codehaus.plexus.classworlds.launcher.Launcher compile ${promptl:=option1} ${promptl:=option2}
```

### External Tools defined

You will see this after you have defined the External Tools: ![External Tools]({{ site.url }}{{ site.baseurl }}/assets/images/external-tools-1.png)

# utPLSQL

In the Java developer community it is best practice to have unit
tests. Unfortunately that is not the case in the Oracle developer
community. Created some 20 years ago by Steven Feuerstein, the third version
of utPLSQL is really very useful. Easy to use and the
[documentation](http://utplsql.org/documentation/) is excellent. It even
integrates with Maven thanks to the
[utPLSQL-maven-plugin](https://github.com/utPLSQL/utPLSQL-maven-plugin). And
there is a plugin too for SQL Developer, see [Running utPLSQL Tests in SQL Developer](https://www.salvis.com/blog/2019/07/06/running-utplsql-tests-in-sql-developer/).

Do not wait any longer, just **unit test** it!


I add unit test code to my packages and the test code is conditional, guarded
by a global configuration package header with some constants defined for
testing and debugging. This way it is easy to de-activate testing code in
production (where utPLQSL should not be installed).
{: .notice--info}

# Perl

A long time favorite since I started developing. Maybe not as sexy as Python
but it works for me. I use it as soon as I need to process Operating System tasks like
working with files and so on.

# Ant

Another long time favorite that I use for executing simple tasks on the Operating
System where dependencies are necessary. I use Ant to export and import Apex
applications. It invokes the Oracle SQLcl client with the appropiate
switches. Ant integrates well with Maven due to the [Apache Maven AntRun Plugin](https://maven.apache.org/plugins/maven-antrun-plugin/).

# DevOps

I have shown you all the tools and techniques you may need plus all the pitfalls
you may encounter while setting up DevOps (or Continuous Integration / Delivery /
Deployment). I have used [Jenkins](https://www.jenkins.io/doc/) in the past.

# Conclusion

This was the final article on "How to build an Oracle Apex application". I
hope to have the inspiration, time and support to write a book about it some day...

I hope you have enjoyed it!

