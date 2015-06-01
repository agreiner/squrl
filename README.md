squrl
========
Syntax and python library for translating database queries encoded in a URL into SQL.

Version 0.5
Annette Greiner, NERSC DAS, 5/22/15

Suppose your web app receives a request for http://mydomain.com/api/uri_fragment
This module handles a uri_fragment, of the form
mytable/expression/PRED[/expression][/CONJ/expression/PRED[/expression]]
where mytable is the table to query, and expression can be of the form
value[/OP/value]
where value can be the name of a column (mycolumn), a column plus an array index (mycolumn.2), a numeric value, or a simple string.
OP (operation) can be DIV, TIMES, PLUS, or MINUS
CONJ (conjunction) can be AND or OR
PRED (predicate) can be EQ, NEQ, LT, LTE, GT, GTE, LIKE, NOTLIKE, ISNULL, or ISNOTNULL
Parentheses can be used at the beginning or end of a value to control the order of operations.
The mytable parameter can also be shorthand for a complex SQL FROM clause, if the clause is entered into the config_queries dict

e.g.
Candidate/DECAM_FLUX.2/GT/0.631
becomes
SELECT * FROM candidate WHERE DECAM_FLUX[2] > 0.631

Candidate/DECAM_FLUX.2/DIV/DECAM_MW_TRANSMISSION.2/GT/0.631/AND/DECAM_FLUX.4/DIV/DECAM_MW_TRANSMISSION.4/GT/DECAM_FLUX.2/DIV/DECAM_MW_TRANSMISSION.2/TIMES/4.365 
becomes
SELECT * FROM candidate WHERE DECAM_FLUX[2] / DECAM_MW_TRANSMISSION[2] > 0.631 AND DECAM_FLUX[4] / DECAM_MW_TRANSMISSION[4] > DECAM_FLUX[2] / DECAM_MW_TRANSMISSION[2] * 4.365

To use SQURL, you'll need to configure the tables that users can query and the limit on rows returned, as below.
in the 'config_queries' dict, enter the name of each table that you want to allow queries against as a key, 
then for each table, 
  - enter a from_clause if you want to specify that portion of the SQL in advance (good for complex queries, such as joins).
  - enter fields as a python list of the columns you want to be queriable. 
  - for db tables that contain arrays, list them under the 'arrays' key. For each, enter a dict with column names as keys and max array indices as values.

    config_queries = {
          'animals': { # this can also be a shortcut name for the particular "from" clause you assign rather than a table name
              'from_clause': 'select * from animals join owners on animals.id=owners.animal_id', # use '' if you just want a simple query of the table
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

