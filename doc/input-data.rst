
.. _input-data:


.. currentmodule:: sasoptpy

Handling Data
=============

*sasoptpy* can work with native Python types and *pandas* objects for all
data operations.
Among *pandas* object types, *sasoptpy* works with :class:`pandas.DataFrame`
and :class:`pandas.Series` objects to construct and manipulate model components.

Indices
-------

Methods like :func:`Model.add_variables` can utilize native Python object
types like list and range as variable and constraint indices.
:class:`pandas.Index` can be used as index as well.

List
~~~~

.. ipython:: python
   :suppress:

   import sasoptpy as so
   so.reset_globals()

.. ipython:: python
   
   m = so.Model(name='demo')
   SEASONS = ['Fall', 'Winter', 'Spring', 'Summer']
   prod_lb = {'Fall': 100, 'Winter': 200, 'Spring': 100, 'Summer': 400}
   production = m.add_variables(SEASONS, lb=prod_lb, name='production')
   print(production)

.. ipython:: python

   print(repr(production['Summer']))

Note that if a list is being used as the index set, associated fields like
`lb`, `ub` should be accesible using the index keys. Accepted types are dict
and pandas.Series.

Range
~~~~~

.. ipython:: python

   link = m.add_variables(range(3), range(2), vartype=so.BIN, name='link')
   print(link)

.. ipython:: python

   print(repr(link[2, 1]))

pandas.Index
~~~~~~~~~~~~

.. ipython:: python

   import pandas as pd
   p_data = [[3, 5, 9],
             [0, -1, 14],
             [5, 6, 20]]
   df = pd.DataFrame(p_data, columns=['c1', 'col_lb', 'col_ub'])
   x = m.add_variables(df.index, lb=df['c1'], vartype=so.INT, name='x')

.. ipython:: python

   print(x)

.. ipython:: python

   df2 = df.set_index([['r1', 'r2', 'r3']])
   y = m.add_variables(df2.index, lb=df2['col_lb'], ub=df2['col_ub'], name='y')

.. ipython:: python

   print(y)

.. ipython:: python

   print(repr(y['r1']))

Set
~~~

*sasoptpy* can work with data on the server and generate abstract
expressions. For this purpose, you can use :class:`Set` objects to represent
PROC OPTMODEL sets.

.. ipython:: python

   m2 = so.Model(name='m2')
   I = m2.add_set(name='I')
   u = m2.add_variables(I, name='u')
   print(I, u)

See :ref:`workflows` for more information on working with server-side models.

Data
----

*sasoptpy* can work with both client-side and server-side data.
Here are some options to load data into the optimization models.


pandas DataFrame
~~~~~~~~~~~~~~~~

:class:`pandas.DataFrame` is the preferred object types when passing data into
*sasoptpy* models.

.. ipython:: python

   data = [
      ['clock', 8, 4, 3],
      ['mug', 10, 6, 5],
      ['headphone', 15, 7, 2],
      ['book', 20, 12, 10],
      ['pen', 1, 1, 15]
      ]
   df = pd.DataFrame(data, columns=['item', 'value', 'weight', 'limit']).set_index(['item'])
   get = so.VariableGroup(df.index, ub=df['limit'], name='get')
   print(get)


Dictionaries
~~~~~~~~~~~~

Lists and dictionaries can be used in expressions and when creating variables.

.. ipython:: python

   items = ['clock', 'mug', 'headphone', 'book', 'pen']
   limits = {'clock': 3, 'mug': 5, 'headphone': 2, 'book': 10, 'pen': 15}
   get2 = so.VariableGroup(items, ub=limits, name='get2')
   print(get2)


CASTable
~~~~~~~~

When a data is available on the server-side, a reference to the object can be
passed. Note that, using CASTable and Abstract Data requires SAS Viya version
3.4.

.. ipython:: python
   :suppress:

   import os
   hostname = os.getenv('CASHOST')
   port = os.getenv('CASPORT')
   from swat import CAS
   session = CAS(hostname, port)


.. ipython:: python

   m2 = so.Model(name='m2', session=session)

.. ipython:: python

   table = session.upload_frame(df)

.. ipython:: python

   print(type(table), table)


.. ipython:: python

   df = pd.DataFrame(data, columns=['item', 'value', 'weight', 'limit'])
   ITEMS, (value, weight, limit) = m2.read_table(df, key=['item'],
      key_type='str', columns=['value', 'weight', 'limit'])
   get3 = m2.add_variables(ITEMS, name='get3')
   print(get3)
   

Abstract Data
~~~~~~~~~~~~~

If you would like to model your problem first and then load data, you can
pass a string for the data sets that will be available later. See following:

.. ipython:: python

   m3 = so.Model(name='m3', session=session)
   ITEMS, (limit) = m3.read_table('DF', key=['item'], key_type=['str'],
      columns=['limit'])
   print(type(ITEMS), ITEMS)

Notice that the key set is created as a reference. We can later solve the
problem after having the data available with the same name, e.g. using the
`upload_frame` function.

.. ipython:: python

   session.upload_frame(df, casout='DF')


Operations
----------

Lists, :class:`pandas.Series`, and :class:`pandas.DataFrame` objects can be
used for mathematical operations like :func:`VariableGroup.mult`.

.. ipython:: python

   sd = [3, 5, 6]
   z = m.add_variables(3, name='z')

.. ipython:: python

   print(z)

.. ipython:: python

   print(repr(z))

.. ipython:: python

   e1 = z.mult(sd)
   print(e1)
   
.. ipython:: python
   
   ps = pd.Series(sd)
   e2 = z.mult(ps)
   print(e2)



