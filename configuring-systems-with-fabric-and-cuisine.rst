Configuring Systems with Fabric and Cuisine
===========================================

:tags: linux, python, fabric
:category: Systems
:author: Trevor
:date: 2015-01-01


I've mentioned Fabric_ before on this blog.  Because so much of my development time is spent in Python, it makes
sense for me to look for system administration tools that are also written in Python.  Fabric fits the bill
perfectly, and allows me to run tasks remotely on multiple machines simultaneously.

.. _Fabric: http://fabric.readthedocs.org/en/1.8/


A useful addition to Fabric is Cuisine_.  **Cuisine is a small set of functions that sit on top of Fabric,
to abstract common administration operations such as file/dir operations, user/group creation,
package install/upgrade, making it easier to write portable administration and deployment scripts.**

.. _Cuisine: https://github.com/sebastien/cuisine
