
Factory Planning 2
==================

Reference
---------

http://go.documentation.sas.com/?docsetId=ormpex&docsetTarget=ormpex_ex4_toc.htm&docsetVersion=14.3&locale=en

http://support.sas.com/documentation/onlinedoc/or/ex_code/143/mpex04.html

Model
-----

.. literalinclude:: ../../examples/factory_planning_2.py

Output
------

.. ipython:: python
   :suppress:
   
   import sasoptpy
   sasoptpy.reset_globals()


.. ipython:: python

   from examples.factory_planning_2 import test
   test(cas_conn)

