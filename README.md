squrl
========
Syntax and python library for translating database queries encoded in a URL (e.g., REST-like queries) into SQL.  
Version 0.5  
Annette Greiner, NERSC DAS, 5/22/15
___

Suppose your web app receives a request for `http://mydomain.com/api/uri_fragment`. This module handles a `uri_fragment`, of the form

        mytable/expression/PRED[/expression][/CONJ/expression/PRED[/expression]]


where mytable is the table to query, and expression can be of the form

 - value[/OP/value]

where 

 - value can be the name of a column (mycolumn), a column plus an array index (mycolumn.2), a numeric value, or a simple string.
 - OP (operation) can be DIV, TIMES, PLUS, or MINUS
 - CONJ (conjunction) can be AND or OR
 - PRED (predicate) can be EQ, NEQ, LT, LTE, GT, GTE, LIKE, NOTLIKE, ISNULL, or ISNOTNULL

Parentheses can be used at the beginning or end of a value to control the order of operations.  
The mytable parameter can also be shorthand for a complex SQL FROM clause, if the clause is entered into the `config_queries` dict.

e.g.

        Candidate/DECAM_FLUX.2/GT/0.631

becomes

        SELECT * FROM candidate WHERE DECAM_FLUX[2] > 0.631

and

        Candidate/FLUX.2/DIV/MW_TRANSMISSION.2/GT/0.631/AND/FLUX.4/DIV/MW_TRANSMISSION.4/GT/FLUX.2/DIV/MW_TRANSMISSION.2/TIMES/4.365 

becomes

        SELECT * FROM candidate WHERE FLUX[2] / MW_TRANSMISSION[2] > 0.631 AND FLUX[4] / MW_TRANSMISSION[4] > FLUX[2] / MW_TRANSMISSION[2] * 4.365

To use SQURL, you'll need to configure the tables that users can query and the limit on rows returned, as below.
in the `config_queries` dict, enter the name of each table that you want to allow queries against as a key, 
then for each table,

  - enter a from_clause if you want to specify that portion of the SQL in advance (good for complex queries, such as joins).
  - enter fields as a python list of the columns you want to be queriable.
  - for db tables that contain arrays, list them under the 'arrays' key. For each, enter a dict with column names as keys and max array indices as values.

Here is an example configuration for a simple query to get all the enabled fields in the query:

        config_queries = {
              'animals': { # table name
                  'from_clause': ''
                  'fields': [
                      'id',
                      'species',
                      'breed',
                      'healthparams'
                  ],
                  'arrays': {
                      {
                      'healthparams': 4
                      }
                  }
              }
              'owners': {
                    ...

Here is a configuration that specifies a custom "from" clause to use with a handle instead of a table name:

        config_queries = {
              'animalsandowners': { # handle for specific from_clause
                  'from_clause': 'select * from animals join owners on animals.id=owners.animal_id',
                  'fields': [
                      'id',
                      'species',
                      'breed',
                      'healthparams'
                  ],
                  'arrays': {
                      {
                      'healthparams': 4
                      }
                  }
              }
              'owners': {
                    ...
