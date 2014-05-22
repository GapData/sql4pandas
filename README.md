pdsql
=====

Efficient SQL bindings for the pandas data analysis library

# Capabilities

## SELECT Statements:
- FROM, WHERE, GROUP BY, ORDER BY Clauses
- LEFT, INNER, RIGHT and OUTER JOINS
- CASE Statements
- Basic functions(ie. SUM, MIN, MAX... works with almost any native pandas aggregate function)
- Standard Comparators (ie. <, >, =, !=, <>), 'AND' and 'OR' to chain
- Comparators efficiently implemented using numexpr, making them faster and more memory efficient than vanilla python
- aliasing for column names

# TODO (POC as of now)
- arithmetic operations(+, -, /, *...etc)
- more functions, such as ISNULL statements
- other statement types such as UPDATE, INSERT, DELETE, SELECT INTO etc
- nested queries
- '?' templating
- performance optimizations
 
# DEPENDENCIES
- pandas 13.0+
- numpy 18.0+
- numexpr
- sqlparse(slightly modified to simplify parsing logic for our use case, custom version included)

# EXAMPLES

    >>> import pandas as pd
    >>> import numpy as np
    >>> from pdsql import PDSQL
    
    >>> tbl1 = pd.DataFrame(np.random.randn(1000, 5) * 50,
                        columns=['a', 'b', 'c', 'd', 'e'])
    >>> tbl2 = tbl1.copy()
    >>> crs = PDSQL({'tbl1': tbl1, 'tbl2': tbl2})
    >>> crs.execute("""SELECT
            CASE
                WHEN SUM(tbl1.e) > 0
                THEN SUM(tbl1.e)
                ELSE SUM(tbl2.a)
            END AS rand,
            MIN(tbl1.b) as min,
            CASE
                WHEN MIN(tbl1.c) < 0
                THEN MIN(tbl1.c)
                WHEN MAX(tbl2.b) > 0
                THEN MAX(tbl1.e)
                ELSE SUM(tbl1.b)
            END as crazy
           FROM tbl2
               LEFT JOIN tbl1
                   ON tbl2.e = tbl1.e
           WHERE tbl1.a > 0 AND tbl2.b < 0
           GROUP BY tbl1.a, tbl2.b
           ORDER BY SUM(tbl1.d)""")
      >>> crs.fetchall()
      
               rand       crazy         min
      
      87    13.980633  -39.880526  -39.880526
      103   23.435746  -18.989008  -18.989008
      166   40.677965  -47.603296  -40.139092
      140   41.618153  -58.673183  -17.608048
      138   20.019576  -40.846443  -14.799018
      136   31.455437  -50.511226   -6.454728
      217   27.721144  -61.249085  -61.249085
      223   57.908348  -32.912267  -32.912267
      207   17.242646   -1.511570  -55.993560
      267    6.517910   -9.434497   -9.434497
      259   18.807235  -98.790074  -81.566930
      9      2.951997  -89.245030  -39.208345
      274  132.999115  -88.597205  -88.597205
      122   28.638471  -91.373880  -50.638201
                  ...         ...         ...
                  
      [277 rows x 3 columns]
                   
