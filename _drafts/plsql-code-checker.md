---
title: A native PL/SQL code checker
categories: development
tags: [ Oracle, APEX, DevOps ]
permalink: /native-plsql-code-checker/
classes: wide
toc: true
toc_label: "Table of contents"
---

This post describes how you can check PL/SQL code using just the information from the Oracle compiler.

# Introduction

In [this article from Steven
Feuerstein](http://stevenfeuersteinonplsql.blogspot.com/2015/04/lint-checkers-for-plsql.html)
a lot of [lint](https://en.wikipedia.org/wiki/Lint_(software)) tools have been
described. Among them the [SonarQube PL/SQL
rules](https://rules.sonarsource.com/plsql), a static code analysis tool, that
seems to be one of the most complete products. However, you need to have a
paid subscription. So, can we just create a simple code checker using Oracle
information alone? Yes, we can!

We can activate PL/SQL warnings and PL/Scope.

# PL/SQL warnings

When you issue this:

```
alter session set PLSQL_WARNINGS = 'ENABLE:ALL';
```

you will get a lot of extra information when you recompile an object.

# PL/Scope

When you read this [AMIS article by Lucas
Jellema](https://technology.amis.nl/it/oracle-11g-generating-plsql-compiler-warnings-java-style-using-plscope/),
the PL/Scope settings enable you to identify (constant) variable declarations,
assignments and references. In the rest of this article I will treat a
constant as a variable.

To get that information you must issue:

```
alter session set PLSCOPE_SETTINGS = 'IDENTIFIERS:ALL';
```

and then recompile your PL/SQL object(s).

## Some code checks

Lucas Jellema identified these:
* Variables that are referenced but never assigned (before that reference)
* Variables that are declared but never used
* Variables that are assigned but never used (after that assignment)

In his article Lucas Jellema used a simple example and did not take into
account a variable having the same name that exists in more than one function
or procedure. So we have to take care of that too.

Besides those mentioned above you can think of these checks too:
* Output parameters should be assigned a value
* Functions should not have output parameters
* Unused procedure and function parameters
* Variables getting out of scope
* Do not define global public variables but use setters and getters

## A more complex example

This package specification should only complain about global public variables:

```
[01] create or replace package ut_code_check_pkg is
[02] 
[03] "abcd" constant varchar2(4 char) := 'abcd';
[04] 
[05] -- Do not defined global public variables but use setters and getters
[06] l_var varchar2(4 char);
[07] 
[08] procedure ut_assign(p_str out nocopy varchar2);
[09] 
[10] procedure ut_reference(p_str in varchar2);
[11] 
[12] end ut_code_check_pkg;
```

This package body should complain about all other checks:

```
[01] create or replace package body ut_code_check_pkg is
[02] 
[03] subtype t_var is varchar2(4 char);
[04] 
[05] procedure ut_assign(p_str out nocopy varchar2)
[06] is
[07] begin
[08]   p_str := "abcd";
[09] end;
[10] 
[11] procedure ut_reference(p_str in varchar2)
[12] is
[13] begin
[14]   if p_str is null
[15]   then
[16]     null;
[17]   end if;
[18] end;
[19] 
[20] procedure ut_var_not_used
[21] is
[22]   l_var t_var;
[23] begin
[24]   null;
[25] end ut_var_not_used;
[26] 
[27] procedure ut_var_assign_declaration
[28] is
[29]   l_var t_var := "abcd";
[30] begin
[31]   null;
[32] end ut_var_assign_declaration;
[33] 
[34] procedure ut_var_assign_direct
[35] is
[36]   l_var t_var;
[37] begin
[38]   l_var := "abcd";
[39] end ut_var_assign_direct;
[40] 
[41] procedure ut_var_assign_indirect
[42] is
[43]   l_var t_var;
[44] begin
[45]   ut_assign(l_var);
[46] end ut_var_assign_indirect;
[47] 
[48] procedure ut_var_assign_after_reference
[49] is
[50]   l_var t_var;
[51] begin
[52]   if l_var is null
[53]   then
[54]     l_var := "abcd";
[55]   end if;
[56] end ut_var_assign_after_reference;
[57] 
[58] -- Output parameters should be assigned a value
[59] -- Unused procedure and function parameters
[60] procedure ut_output_parameters_not_set(p_i in varchar2, p_io in out varchar2, p_o out varchar2)
[61] is
[62] begin
[63]   null;
[64] end;
[65] 
[66] -- Functions should not have output parameters
[67] -- Unused procedure and function parameters
[68] function ut_function_output_parameters(p_i in varchar2, p_io in out varchar2, p_o out varchar2)
[69] return varchar2
[70] is
[71] begin
[72]   return null;
[73] end;
[74] 
[75] -- Variables getting out of scope
[76] procedure ut_variables_out_of_scope
[77] is
[78]   i_idx integer;
[79] begin
[80]   for i_idx in 1..2
[81]   loop
[82]     null;
[83]   end loop;
[84] 
[85]   declare
[86]     i_idx exception;
[87]   begin
[88]     null;
[89]   end;
[90] end;
[91] 
[92] end ut_code_check_pkg;
```

## How does PL/Scope store the information?

The following query on the Oracle dictionary view USER_IDENTIFIERS:

```
select  line
,       name
,       type
,       usage
,       usage_id
,       usage_context_id
from    user_identifiers 
where   object_name like 'UT_CODE_CHECK_PKG'
and     object_type = 'PACKAGE BODY'
order by
        line
,       usage_id
```

gives this result set:

|LINE|NAME|TYPE|USAGE|USAGE_ID|USAGE_CONTEXT_ID|
|---:|:---|:---|:----|-------:|---------------:|
|01|UT_CODE_CHECK_PKG|PACKAGE|DEFINITION|1|0|
|03|T_VAR|SUBTYPE|DECLARATION|2|1|
|03|VARCHAR2|CHARACTER DATATYPE|REFERENCE|3|2|
|05|UT_ASSIGN|PROCEDURE|DEFINITION|4|1|
|05|P_STR|FORMAL OUT|DECLARATION|5|4|
|05|VARCHAR2|CHARACTER DATATYPE|REFERENCE|6|5|
|08|P_STR|FORMAL OUT|ASSIGNMENT|7|4|
|08|abcd|CONSTANT|REFERENCE|8|7|
|11|UT_REFERENCE|PROCEDURE|DEFINITION|9|1|
|11|P_STR|FORMAL IN|DECLARATION|10|9|
|11|VARCHAR2|CHARACTER DATATYPE|REFERENCE|11|10|
|14|P_STR|FORMAL IN|REFERENCE|12|9|
|20|UT_VAR_NOT_USED|PROCEDURE|DECLARATION|13|1|
|20|UT_VAR_NOT_USED|PROCEDURE|DEFINITION|14|13|
|22|L_VAR|VARIABLE|DECLARATION|15|14|
|22|T_VAR|SUBTYPE|REFERENCE|16|15|
|27|UT_VAR_ASSIGN_DECLARATION|PROCEDURE|DECLARATION|17|1|
|27|UT_VAR_ASSIGN_DECLARATION|PROCEDURE|DEFINITION|18|17|
|29|L_VAR|VARIABLE|DECLARATION|19|18|
|29|T_VAR|SUBTYPE|REFERENCE|20|19|
|29|L_VAR|VARIABLE|ASSIGNMENT|21|19|
|29|abcd|CONSTANT|REFERENCE|22|21|
|34|UT_VAR_ASSIGN_DIRECT|PROCEDURE|DECLARATION|23|1|
|34|UT_VAR_ASSIGN_DIRECT|PROCEDURE|DEFINITION|24|23|
|36|L_VAR|VARIABLE|DECLARATION|25|24|
|36|T_VAR|SUBTYPE|REFERENCE|26|25|
|38|L_VAR|VARIABLE|ASSIGNMENT|27|24|
|38|abcd|CONSTANT|REFERENCE|28|27|
|41|UT_VAR_ASSIGN_INDIRECT|PROCEDURE|DECLARATION|29|1|
|41|UT_VAR_ASSIGN_INDIRECT|PROCEDURE|DEFINITION|30|29|
|43|L_VAR|VARIABLE|DECLARATION|31|30|
|43|T_VAR|SUBTYPE|REFERENCE|32|31|
|45|UT_ASSIGN|PROCEDURE|CALL|33|30|
|45|L_VAR|VARIABLE|REFERENCE|34|33|
|48|UT_VAR_ASSIGN_AFTER_REFERENCE|PROCEDURE|DECLARATION|35|1|
|48|UT_VAR_ASSIGN_AFTER_REFERENCE|PROCEDURE|DEFINITION|36|35|
|50|L_VAR|VARIABLE|DECLARATION|37|36|
|50|T_VAR|SUBTYPE|REFERENCE|38|37|
|52|L_VAR|VARIABLE|REFERENCE|39|36|
|54|L_VAR|VARIABLE|ASSIGNMENT|40|36|
|54|abcd|CONSTANT|REFERENCE|41|40|
|60|UT_OUTPUT_PARAMETERS_NOT_SET|PROCEDURE|DECLARATION|44|1|
|60|UT_OUTPUT_PARAMETERS_NOT_SET|PROCEDURE|DEFINITION|45|44|
|60|P_I|FORMAL IN|DECLARATION|46|45|
|60|VARCHAR2|CHARACTER DATATYPE|REFERENCE|47|46|
|60|P_IO|FORMAL IN OUT|DECLARATION|48|45|
|60|VARCHAR2|CHARACTER DATATYPE|REFERENCE|49|48|
|60|P_O|FORMAL OUT|DECLARATION|50|45|
|60|VARCHAR2|CHARACTER DATATYPE|REFERENCE|51|50|
|68|UT_FUNCTION_OUTPUT_PARAMETERS|FUNCTION|DECLARATION|52|1|
|68|UT_FUNCTION_OUTPUT_PARAMETERS|FUNCTION|DEFINITION|53|52|
|68|P_I|FORMAL IN|DECLARATION|54|53|
|68|VARCHAR2|CHARACTER DATATYPE|REFERENCE|55|54|
|68|P_IO|FORMAL IN OUT|DECLARATION|56|53|
|68|VARCHAR2|CHARACTER DATATYPE|REFERENCE|57|56|
|68|P_O|FORMAL OUT|DECLARATION|58|53|
|68|VARCHAR2|CHARACTER DATATYPE|REFERENCE|59|58|
|69|VARCHAR2|CHARACTER DATATYPE|REFERENCE|60|53|
|76|UT_VARIABLES_OUT_OF_SCOPE|PROCEDURE|DECLARATION|61|1|
|76|UT_VARIABLES_OUT_OF_SCOPE|PROCEDURE|DEFINITION|62|61|
|78|I_IDX|VARIABLE|DECLARATION|63|62|
|78|INTEGER|SUBTYPE|REFERENCE|64|63|
|80|I_IDX|ITERATOR|DECLARATION|65|62|
|86|I_IDX|EXCEPTION|DECLARATION|66|62|


Some constructs:

### Function/procedure declarations and definitions

You can see from the identifier output that the global UT_ASSIGN and
UT_REFERENCE procedures are not declared in the package body but just defined
unlike all other local functions and procedures. That is logical too since
they are global and thus declared in the package specification.

### An uninitialized variable

On line 22 you will find a variable "L_VAR" that is not initialized.

### An initialized variable

On line 29 you will find a variable "L_VAR" that is initialized and it has a
USAGE_CONTEXT_ID equal to the USAGE_ID of the procedure definition
"UT_VAR_ASSIGN_DECLARATION". As you can see the assignment on the same line
has as USAGE_CONTEXT_ID the USAGE_ID of the variable.

### An assignment later on (after the declaration)

On line 38 the reference to "L_VAR" has a USAGE_CONTEXT_ID of 24, being the
USAGE_ID of procedure definition "UT_VAR_ASSIGN_DIRECT".

### An initialization thru another procedure

In line 45 the variable is initialized by another procedure. Given the fact
that this procedure is defined in the same unit and it has just one OUT
parameter, it is possible to determine that the variable is initialized. However
that can be difficult in other circumstances so I will ignore this.

# What checks can be performed by PL/Scope?

Below the various categories are mentioned. Please note that a variable may
belong to more than one category. For example the variable declared in line 36
is declared but never used but it has also been assigned a value that is never
used after that assignment.

## Identifiers that are declared but never used

I will not only consider variables (or constants) but any identifier type
except specification, body, function or procedure (already covered by PL/SQL
warnings). After all, it is nice to know which cursors, exceptions, etcetera
are not used as well.

When you have an identifier that is not a declaration, the idea is to find its
(nearest) declaration scope (in the innermost function/procedure, then its
enclosing function/procedure if any, up till the outermost
body/function/procedure). If you have done that for all non-declarations you
can subtract their scoped declarations from all the declared identifiers and
then you have the unused declarations.

For example in procedure "UT_VAR_NOT_USED", the declaration of reference
"T_VAR" is not found in the procedure itself but in the package body. On the
other hand the declaration of "L_VAR" has no references. Hence "T_VAR" will
not be listed as unused but "L_VAR" will.

Please note that this check is never performed for a package specification
because those variables will normally be used in its body or in another
object.

## Variables that are referenced but never assigned a value (before that reference)

If there is no assignment between the declaration and the first reference then
such a variable falls into this category.

Please note that this check is never performed for a package (or object type)
specification.  And this check is less usefull if the scopes of declaration
and reference are different. This check will thus performed only if the
declaration and reference are in the same scope.

## Variables that are assigned a value but never used (after that assignment)

If there is a reference before but **not** after the last assignment, then
such a variable falls into this category. if there would not have been a
reference before the last assignment neither then it falls into the first
category of unused variables because there are zero references.

Please note that this check is never performed for a package (or object type)
specification.  And this check is less usefull if the scopes of declaration
and assignment are different. This check will thus performed only if the
declaration and assignment are in the same scope.

## Output parameters should be assigned a value

An item of TYPE "FORMAL IN OUT" or "FORMAL OUT" must be assigned a value later
on. The USAGE_CONTEXT_ID of the parameter is the USAGE_ID of the procedure
definition.

## Functions should not have output parameters

This is considered bad practice. An item of TYPE "FORMAL IN OUT" or "FORMAL
OUT" must not be part of a function.

## Unused procedure and function parameters

A parameter (TYPE "FORMAL IN", "FORMAL IN OUT" or "FORMAL OUT") is never
referenced nor assigned as value.

## Variables getting out of scope

When a function/procedure declares a variable and a FOR LOOP uses an iterator
with the same name, the original variable is out of scope. The same
when a new DECLARE block declare a variable with the same name.

This is detected by having the same NAME with TYPE "DECLARATION" and the same USAGE_CONTEXT_ID.

## Do not define global public variables but use setters and getters

It is considered bad practice to define a variable in in package
specification. Use setters and getters instead.

Please note that this check is **only** performed for a package specification.

# Example queries

## Identifiers that are declared but never used

The following query lists the unused identifiers:

```
[01] with src as (
[02]   select  line
[03]   ,       name
[04]   ,       type
[05]   ,       usage
[06]   ,       usage_id
[07]   ,       usage_context_id
[08]   from    user_identifiers 
[09]   where   object_name = 'UT_CODE_CHECK_PKG'
[10]   and     object_type = 'PACKAGE BODY'
[11]   ), identifiers as (
[12]   select  src.*
[13]   ,       rtrim(replace(sys_connect_by_path(case when usage = 'DEFINITION' then name || '.' end, '|'), '|'), '.') as name_scope
[14]   ,       rtrim(replace(sys_connect_by_path(case when usage = 'DEFINITION' then usage_id || '.' end, '|'), '|'), '.') as usage_id_scope
[15]   from    src
[16]   start with
[17]           usage_id = 1
[18]   connect by  
[19]           usage_context_id = prior usage_id
[20] )
[21] , declarations as (
[22]   select  *
[23]   from    identifiers
[24]   where   usage = 'DECLARATION'
[25] )
[26] , non_declarations as (
[27]   select  i.*
[28]   ,       di.usage_id_scope as declaration_usage_id_scope
[29]   ,       row_number() over (partition by i.name, i.type, i.usage_id order by length(di.usage_id_scope) desc) as seq -- longest usage_id_scope first, i.e. prefer innermost declaration
[30]   from    identifiers i
[31]           left outer join declarations di
[32]           on di.name = i.name and di.type = i.type and i.usage_id_scope like di.usage_id_scope || '%'
[33]   where   i.usage != 'DECLARATION'        
[34] )
[35] , unused_identifiers as (
[36]   select  d.*
[37]   from    declarations d
[38]           left outer join non_declarations nd
[39]           -- skip assignments to a variable/constant but not for instance to a parameter
[40]           on nd.seq = 1 and not(nd.usage = 'ASSIGNMENT' and nd.type in ('VARIABLE', 'CONSTANT')) and nd.name = d.name and nd.type = d.type and nd.declaration_usage_id_scope = d.usage_id_scope
[41]   where   d.type not in ('PACKAGE BODY', 'FUNCTION', 'PROCEDURE')
[42]   and     nd.name is null
[43] )
[44] select  *
[45] from    unused_identifiers
[46] order by
[47]         line
```

Some explanation:

| Line(s) | Remark |
| :------ | :----- |
| 13-14   | The name scope will construct something like &lt;package body&gt;[.&lt;function/procedure&gt;]. Please note that sys_connect_by_path needs a non-null value as 2nd parameter but that needs to be stripped later on. The usage_id_scope is the really unique scope path (Oracle allows overloading for instance so function/procedure names can be the same but not the signature). |
| 17 | We must start with the package definition (body) usage id. |
| 29,32 | We want the declaration nearest to us (so preferable in the same function/procedure and finally down to the package body) |
| 40 | We need to use seq = 1 to get the innermost declaration and we want to skip assignments to a constant/variable (but not to a parameter for instance) |
| 41 | We are not interested in package body/function/procedure declarations |
| 42 | Assuring that nd.name is null means that the declaration identifier is not used anywhere |

This is the result set:

|LINE|NAME|TYPE|USAGE|USAGE_ID|USAGE_CONTEXT_ID|NAME_SCOPE|USAGE_ID_SCOPE|
|---:|:---|:---|:----|-------:|---------------:|:---------|:-------------|
|22|L_VAR|VARIABLE|DECLARATION|15|14|UT_CODE_CHECK_PKG.UT_VAR_NOT_USED|1.14|
|29|L_VAR|VARIABLE|DECLARATION|19|18|UT_CODE_CHECK_PKG.UT_VAR_ASSIGN_DECLARATION|1.18|
|36|L_VAR|VARIABLE|DECLARATION|25|24|UT_CODE_CHECK_PKG.UT_VAR_ASSIGN_DIRECT|1.24|
|60|P_IO|FORMAL IN OUT|DECLARATION|46|43|UT_CODE_CHECK_PKG.UT_OUTPUT_PARAMETERS_NOT_SET|1.43|
|60|P_O|FORMAL OUT|DECLARATION|48|43|UT_CODE_CHECK_PKG.UT_OUTPUT_PARAMETERS_NOT_SET|1.43|
|60|P_I|FORMAL IN|DECLARATION|44|43|UT_CODE_CHECK_PKG.UT_OUTPUT_PARAMETERS_NOT_SET|1.43|
|68|P_O|FORMAL OUT|DECLARATION|56|51|UT_CODE_CHECK_PKG.UT_FUNCTION_OUTPUT_PARAMETERS|1.51|
|68|P_I|FORMAL IN|DECLARATION|52|51|UT_CODE_CHECK_PKG.UT_FUNCTION_OUTPUT_PARAMETERS|1.51|
|68|P_IO|FORMAL IN OUT|DECLARATION|54|51|UT_CODE_CHECK_PKG.UT_FUNCTION_OUTPUT_PARAMETERS|1.51|
|78|I_IDX|VARIABLE|DECLARATION|61|60|UT_CODE_CHECK_PKG.UT_VARIABLES_OUT_OF_SCOPE|1.60|
|80|I_IDX|ITERATOR|DECLARATION|63|60|UT_CODE_CHECK_PKG.UT_VARIABLES_OUT_OF_SCOPE|1.60|
|86|I_IDX|EXCEPTION|DECLARATION|64|60|UT_CODE_CHECK_PKG.UT_VARIABLES_OUT_OF_SCOPE|1.60|

It seems to work:
* "L_VAR" is reported as unused in those procedures where it is not referenced.
* The parameters "P_I", "P_IO" and "P_O" that are not assigned a value nor referenced are reported.
* The "I_IDX" identifiers in the procedure "UT_VARIABLES_OUT_OF_SCOPE" are reported too.

# Conclusion

Oracle gives use some basic information about compiler errors and
warnings. The queries to implement those checks are not always easy to
construct and it seems advisable to have them available as a table function
and not just as a SQL script. That way the checks can be issued not only from
an IDE but also from a Continuous Integration environment. That may be an argument for not 
using another code checker that only is available thru an IDE. 
