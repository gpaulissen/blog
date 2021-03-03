---
title: How to build an Oracle APEX application (6)
categories: development
tags: [ Oracle, APEX, DevOps, SqlDeveloper, utPLSQL, SonarQube, Perl, Ant, DevOps ]
permalink: /oracle-apex-how-to-build-6/
toc: true
toc_label: "Table of contents"
toc_icon: "database"
excerpt: "This last article: Oracle SQL Developer, utPLSQL, SonarQube, Perl, Ant and DevOps."
last_modified_at: 2021-03-03T15:21:00
---

<figure class="centered">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/512px-Devops-toolchain.svg.png" alt="">
	<figcaption>Kharnagy, CC BY-SA 4.0 <https://creativecommons.org/licenses/by-sa/4.0>, via Wikimedia Commons</figcaption>
</figure>

Last time in ["How to build an Oracle APEX application (5)"]({{ site.url }}{{
site.baseurl }}/oracle-apex-how-to-build-5/), I told you about Git, Subversion, Maven and Flyway.

In this final article, I will discuss the following tools & methods: Oracle SQL Developer,
utPLSQL, SonarCube, Perl, Ant and DevOps.

# Oracle SQL Developer

> [Oracle SQL Developer](https://www.oracle.com/database/technologies/appdev/sqldeveloper-landing.html)
> is a free, integrated development environment that simplifies the development
> and management of Oracle Database in both traditional and Cloud
> deployments. SQL Developer offers complete end-to-end development of your
> PL/SQL applications, a worksheet for running queries and scripts, a DBA
> console for managing the database, a reports interface, a complete data
> modeling solution, and a migration platform for moving your 3rd party
> databases to Oracle.

I use Oracle SQL Developer as my main database development environment but I
will only discuss its External Tools section here because I needed that to
launch Maven builds on a Citrix server. There the only way to launch Maven was
via Oracle SQL Developer and the Java executable. The command line was
forbidden, a security measure I can understand and approve of but as a developer
I needed a way to launch the installation of my applications.

In the examples I assume that an environment variable `MAVEN_HOME` is defined that points
to the Maven home environment. 

## External tools

The External Tools section allows you to define your own tool (command) that can be
invoked either contextual (right click in an editor) or from a menu.

Go to the `Tools` menu and click `External Tools...`: ![External Tools menu]({{ site.url }}{{ site.baseurl
}}/assets/images/external-tools-0.png)

### The tool Maven scm:update install

This command is used to update a project from a source repository and install
the package contents created by Maven in the local Maven repository. I have a Maven Oracle
build project stored in a (private) GitHub source repository that is used by
all other Maven Oracle application projects. This Maven Oracle build project
needs to be installed (locally) before it can be referenced as a dependency by
an application project.

I intend to create a public open-source Maven Oracle build project later
on. This project contains:
- a Maven POM file for installing database migrations via Flyway
- a Maven POM file for exporting and importing APEX applications
- Ant and Perl scripts to support the above functionality

To create this external tool use executable `java` and these arguments:

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


### The tool Maven db

This command installs the database migration scripts using Flyway.

As you may remember from ["How to build an Oracle APEX application (2)"]({{ site.url }}{{
site.baseurl }}/oracle-apex-how-to-build-2/) the project layout has a db
folder below the project root folder. Hence the invocation of Maven is 1 level lower than the root as you can see
from `-Dmaven.multiModuleProjectDirectory` in the following arguments:

```
-classpath "${env:var=MAVEN_HOME}\boot\plexus-classworlds-2.6.0.jar" "-Dclassworlds.conf=${env:var=MAVEN_HOME}\bin\m2.conf" "-Dmaven.home=${env:var=MAVEN_HOME}" "-Dlibrary.jansi.path=${env:var=MAVEN_HOME}\lib\jansi-native" "-Dmaven.multiModuleProjectDirectory=${file.dir}/.." org.codehaus.plexus.classworlds.launcher.Launcher compile flyway:repair flyway:migrate ${promptl:=option1} ${promptl:=option2}
```

### The tool Maven apex

This command allows you to export or import an APEX application.

Same level as db since the apex folder has the same height as db, but other goals:

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
production (where utPLSQL should not be installed).
{: .notice--info}

The following sections show an example.

## Configuration package

```
create or replace package cfg_pkg
is

c_debugging constant boolean := false;
  
c_testing constant boolean := true;

end cfg_pkg;
```

## A package specification with a function/procedure to unit test

A unit test procedure must be prefixed by:

```
--%test
```

as you can see below:


```
create or replace package data_api_pkg authid current_user is

/**
 * Raise a generic application error.
 *
 * @param p_error_code  For instance a business rule
 * @param p_p1          Parameter 1
 * @param p_p2          Parameter 2
 * @param p_p3          Parameter 3
 * @param p_p4          Parameter 4
 * @param p_p5          Parameter 5
 * @param p_p6          Parameter 6
 * @param p_p7          Parameter 7
 * @param p_p8          Parameter 8
 * @param p_p9          Parameter 9
 */
procedure raise_error
( p_error_code in varchar2
, p_p1 in varchar2 default null
, p_p2 in varchar2 default null
, p_p3 in varchar2 default null
, p_p4 in varchar2 default null
, p_p5 in varchar2 default null
, p_p6 in varchar2 default null
, p_p7 in varchar2 default null
, p_p8 in varchar2 default null
, p_p9 in varchar2 default null
);

$if cfg_pkg.c_testing $then

--%suite

--%test
procedure ut_raise_error;

$end

end data_api_pkg;
```

You can use any name you like for the unit test procedure but adding `ut_` as
prefix is the convention of the previous versions of utPLSQL.

## A package body with a function/procedure to unit test

```
create or replace package body data_api_pkg
is

procedure raise_error
( p_error_code in varchar2
, p_p1 in varchar2 default null
, p_p2 in varchar2 default null
, p_p3 in varchar2 default null
, p_p4 in varchar2 default null
, p_p5 in varchar2 default null
, p_p6 in varchar2 default null
, p_p7 in varchar2 default null
, p_p8 in varchar2 default null
, p_p9 in varchar2 default null
)
is
  l_p varchar2(32767);
  l_error_message varchar2(2000) := "#" || p_error_code;
begin
$if cfg_pkg.c_debugging $then
  dbug.enter($$PLSQL_UNIT || '.RAISE_ERROR');
  dbug.print
  ( dbug."input"
  , 'p_error_code: %s; p_p1: %s; p_p2: %s; p_p3: %s; p_p4: %s'
  , p_error_code
  , p_p1
  , p_p2
  , p_p3
  , p_p4
  );
  dbug.print
  ( dbug."input"
  , 'p_p5: %s; p_p6: %s; p_p7: %s; p_p8: %s; p_p9: %s'
  , p_p5
  , p_p6
  , p_p7
  , p_p8
  , p_p9
  );
$end

  if p_error_code is null
  then
    raise value_error;
  end if;

  <<append_loop>>
  for i_idx in 1..9
  loop
    l_p :=
      case i_idx
        when 1 then p_p1
        when 2 then p_p2
        when 3 then p_p3
        when 4 then p_p4
        when 5 then p_p5
        when 6 then p_p6
        when 7 then p_p7
        when 8 then p_p8
        when 9 then p_p9
      end;

    l_error_message := l_error_message || "#" || l_p;
  end loop append_loop;

  -- strip empty parameters from the end
  l_error_message := rtrim(l_error_message, '#');

  raise_application_error(c_exception, l_error_message);
  
$if cfg_pkg.c_debugging $then
  dbug.leave;
exception
  when others
  then
    dbug.leave_on_error;
    raise;
$end  
end raise_error;

$if cfg_pkg.c_testing $then

procedure ut_raise_error
is
  l_msg_exp varchar2(4000 char);
begin
  for i_idx in 1..6
  loop
    begin
      case i_idx
        when 1
        then l_msg_exp := '#'; raise_error(null);             
        when 2
        then l_msg_exp := '#abc'; raise_error('abc');
        when 3
        then l_msg_exp := '#def#p1'; raise_error('def', p_p1 => 'p1');
        when 4
        then l_msg_exp := '#ghi##p2'; raise_error('ghi', p_p2 => 'p2');
        when 5
        then l_msg_exp := '#jkl#########p9'; raise_error('jkl', p_p9 => 'p9');
        when 6
        then l_msg_exp := '#MNO#a#b#c#d#e#f#g#h#i'; raise_error('MNO', p_p1 => 'a', p_p2 => 'b', p_p3 => 'c', p_p4 => 'd', p_p5 => 'e', p_p6 => 'f', p_p7 => 'g', p_p8 => 'h', p_p9 => 'i');
      end case;
      raise program_error;
    exception
      when others
      then
        case i_idx
          when 1
          then
            ut.expect(sqlcode, 'sqlcode '|| i_idx).to_equal(-6502);
            
          else
            ut.expect(sqlcode, 'sqlcode '|| i_idx).to_equal(c_exception);
            ut.expect(sqlerrm, 'sqlerrm '|| i_idx).to_be_like('%'||l_msg_exp||'%');
        end case;
    end;
  end loop;
end ut_raise_error;

$end

end data_api_pkg;
```

# SonarQube

As shown by the [SonarQube PL/SQL rules](https://rules.sonarsource.com/plsql)
this static code analysis tool improves the quality of code. It is a
[lint](https://en.wikipedia.org/wiki/Lint_(software)) like tool. Other PL/SQL
lint checkers are described in [this article from Steven
Feuerstein](http://stevenfeuersteinonplsql.blogspot.com/2015/04/lint-checkers-for-plsql.html).

Here are some rules:
- Inserts should include values for non-null columns
- Predefined exceptions should not be overridden

But I really question this rule:
- Quoted identifiers should not be used

SonarQube allows you to define [your own rules](https://docs.sonarqube.org/latest/extend/adding-coding-rules/) and can be run by Maven using [SonarScanner for Maven](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-maven/).

# Perl

A long time personal favorite since I started developing. Maybe not as sexy as Python
but it works for me. I use it as soon as I need to process Operating System tasks like
working with files and so on.

Some people may prefer Bash (Unix/Linux) or Powershell (Windows) scripting but
I like my development tools to be platform independent so I can use them in
any project.

# Ant

Another long time favorite that I use for executing simple tasks on the Operating
System where dependencies are necessary. I use Ant to export and import APEX
applications. It invokes the [Oracle SQLcl client](https://www.oracle.com/database/technologies/appdev/sqlcl.html) with the appropiate
switches. Ant integrates well with Maven due to the [Apache Maven AntRun Plugin](https://maven.apache.org/plugins/maven-antrun-plugin/).

> SQLcl is a new Java-based command-line interface for Oracle Database.
>
> <cite><a href="https://blogs.oracle.com/oraclemagazine/the-modern-command-line">By Jeff Smith, September/October 2015</a></cite>

# DevOps

> DevOps is a set of practices that combines software development (Dev) and IT
> operations (Ops). It aims to shorten the systems development life cycle and
> provide continuous delivery with high software quality. DevOps is
> complementary with Agile software development; several DevOps aspects came
> from the Agile methodology.
>
> <cite><a href="https://en.wikipedia.org/wiki/DevOps">DevOps, Wikipedia</a></cite>

> As DevOps is intended to be a cross-functional mode of working, those who
> practice the methodology use different sets of tools - referred to as
> "toolchains" - rather than a single one. These toolchains are expected to
> fit into one or more of the following categories, reflective of key aspects
> of the development and delivery process:
>
> - Coding - code development and review, source code management tools, code merging.
> - Building - continuous integration tools, build status.
> - Testing - continuous testing tools that provide quick and timely feedback on business risks.
> - Packaging - artifact repository, application pre-deployment staging.
> - Releasing - change management, release approvals, release automation.
> - Configuring - infrastructure configuration and management, infrastructure as code tools.
> - Monitoring - applications performance monitoring, end-user experience.
>
> Some categories are more essential in a DevOps toolchain than others;
> especially continuous integration (e.g. Jenkins, Gitlab, Bitbucket
> pipelines) and infrastructure as code (e.g., Terraform, Ansible, Puppet).
>
> <cite><a href="https://en.wikipedia.org/wiki/DevOps">DevOps, Wikipedia</a></cite>

I have shown you all the tools and techniques you may need plus all the pitfalls
you may encounter while setting up DevOps (or Continuous Integration / Delivery /
Deployment). I have used [Jenkins](https://www.jenkins.io/doc/) in the past.

The following table shows the match between DevOps processes and tools:

| DevOps process | tools |
| ============== | ===== |
| Coding      	 | PL/SQL, front-end like APEX, SQL Developer (Data Modeler), Ant, Perl |
| Building    	 | Maven (using Flyway, Ant) |
| Testing     	 | Maven (using utPLSQL, SonarQube) |
| Packaging   	 | Maven (using [Artifactory](https://jfrog.com/artifactory/), [Nexus](https://www.sonatype.com/nexus/repository-pro) or GitHub) |
| Releasing   	 | N/A |
| Configuring 	 | N/A |
| Monitoring 		 | N/A |

Please note that I have not mentioned Jenkins as tool above. Jenkins is the tool
to invoke Maven on a remote integration server. Locally you do not need Jenkins: you just run
the appropiate Maven commands from the command line.
{: .notice--info}

An important aspect that I may not have mentioned before is that a **Maven build**
**fails** as soon there is an **error**. The **same** is true for a **Jenkins build** that
fails if one of its build steps fails.
{: .notice--info}

# Conclusion

This was the final article on "How to build an Oracle APEX application". I
hope to have the inspiration, time and support to write a book about it some day...

I hope you have enjoyed it!

