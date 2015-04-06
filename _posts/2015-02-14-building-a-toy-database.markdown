---
layout: post
title:  "Building a toy database"
date:   2013-9-19
comments: True
categories: C++
---

As in the title, this a toy implementation of a relational database
([RDBMS](http://en.wikipedia.org/wiki/Relational_database_management_system)) and is not supposed
to be used on production systems. That said, let's start.

## Requirements

We're gonna keep it simple:

    - Store the schema somewhere
    - Store the data related to the schemes somewhere
    - Work with strings only
    - Support only a subset of SQL

## Storing the schema

Our schema will consist by the declaration of the database, its tables and fields.
The definition will be as follows:

    #> CREATE DB my_database
    #> CREATE TABLE my_table (row_1, row_2, row_3)

Since we only accept strings to store there's no need for a type definition.

Now that we defined how it's gonna work, let's implement it.

Starting with the database creation, we'll create a file on the filesystem for each created
database. This file must have the tables declarations on it. Its structure will consist of the following:

    # file my_database
    my_table[col_1, col_2, col_3] = {[(row 1 col 1 in table my_table), (row 1 col 2 in table my_table), (row 1 col 3 in table my_table)], [(row 2 col 1 in table my_table), (row 2 col 2 in table my_table), (row 2 col 3 in table my_table)]}

Explain this shit here! Let's jump into the implementation:


