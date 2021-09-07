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

In [this article from Steven Feuerstein](http://stevenfeuersteinonplsql.blogspot.com/2015/04/lint-checkers-for-plsql.html) a
lot of [lint](https://en.wikipedia.org/wiki/Lint_(software)) tools have been described. Among them the [SonarQube PL/SQL rules](https://rules.sonarsource.com/plsql), 
a static code analysis tool, that seems to be one of the most complete products. However, you need to have a paid subscription. Is there any way we can just create a simple code checker using Oracle information alone? Yes, we can! 

# PL/Scope

When you read this [AMIS article by Lucas Jellema](https://technology.amis.nl/it/oracle-11g-generating-plsql-compiler-warnings-java-style-using-plscope/), the PL/Scope settings enable you to identify (constant) variable declarations, assignments and references. In the rest of this article I will treat a constant as a variable.

To get that information you must issue:

```
alter session set plscope_settings = 'IDENTIFIERS:ALL';
```

and then recompile your PL/SQL object(s).

## Some code checks

Lucas Jellema identified these:
<ul>
<li>Variables that are referenced but never assigned (before that reference)</li>
<li>Variables that are declared but never used</li>
<li>Variables that are assigned but never used (after that assignment)</li>
</ul>

In his article Lucas Jellema used a simple example and did not take into
account a variable having the same name that exists in more than one function
or procedure. So we have to take care of that too.

Besides those mentioned above you can think of these checks too:
<ul>
<li>Output parameters should be assigned a value</li>
<li>Functions should not have output parameters</li>
<li>Unused procedure and function parameters</li>
<li>Variables getting out of scope</li>
<li>Do not defined global public variables but use setters and getters</li>
</ul>

## A more complex example

This package specification should only complain about global public variables:

```
[1] create or replace package ut_code_check_pkg is
[2] 
[3] "abcd" constant varchar2(4 char) := 'abcd';
[4] 
[5] -- Do not defined global public variables but use setters and getters
[6] l_var varchar2(4 char);
[7] 
[8] end ut_code_check_pkg;
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

## How to implement the native PL/SQL code checker?

Below the various categories are mentioned. Please note that a variable may
belong to more than one category. For example the variable declared in line 36
is declared but never used but it has also been assigned a value that is never
used after that assignment.

### Variables that are declared but never used

This is quite simple: for every declaration of variable v and USAGE_CONTEXT_ID
u (the USAGE_ID of its defining function/procedure), there is no reference for
the same variable v and USAGE_CONTEXT_ID u. References in other procedures are thus
not taken into account.

Please note that this check is never performed for a package (or object type) specification.

### Variables that are referenced but never assigned a value (before that reference)

If there is no assignment between the declaration and the first reference then
such a variable falls into this category.

Please note that this check is never performed for a package (or object type) specification.

### Variables that are assigned a value but never used (after that assignment)

If there is a reference before but **not** after the last assignment, then
such a variable falls into this category. if there would not have been a
reference before the last assignment neither then it falls into the first
category of unused variables because there are zero references.

Please note that this check is never performed for a package (or object type) specification.

### Output parameters should be assigned a value

An item of TYPE "FORMAL IN OUT" or "FORMAL OUT" must be assigned a value later
on. The USAGE_CONTEXT_ID of the parameter is the USAGE_ID of the procedure
definition.

### Functions should not have output parameters

This is considered bad practice. An item of TYPE "FORMAL IN OUT" or "FORMAL
OUT" must not be part of a function.

### Unused procedure and function parameters

A parameter (TYPE "FORMAL IN", "FORMAL IN OUT" or "FORMAL OUT") is never
referenced nor assigned as value.

### Variables getting out of scope

When a function/procedure declares a variable and a FOR LOOP uses an iterator
with the same name, the original variable is out of scope. The same
when a new DECLARE block declare a variable with the same name.

This is detected by having the same NAME with TYPE "DECLARATION" and the same USAGE_CONTEXT_ID.

### Do not defined global public variables but use setters and getters

It is considered bad practice to define a variable in in package
specification. Use setters and getters instead.

Please note that this check is **only** performed for a package specification.
