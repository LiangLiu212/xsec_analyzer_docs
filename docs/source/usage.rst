Quick Start
===========

.. _installation:

Installation
------------

The `xsec-analyzer <https://github.com/LiangLiu212/xsec_analyzer/tree/docs>`_ repository contains a set of Bash scripts, ROOT macros, and \cpp\ source files intended for interactive use on the MicroBooNE gpvm nodes. To install the code, navigate to a writeable area in your

.. code-block:: console

   $ git clone git@github.com:LiangLiu212/xsec_analyzer.git NAME
   

where ``NAME`` should be replaced with the desired name of the folder that
will hold the clone of the main Git repository. Once the cloning process is complete, you may set up the runtime environment (ROOT, etc.) and compile `xsec-analyzer <https://github.com/LiangLiu212/xsec_analyzer/tree/docs>`_ by executing

.. code-block:: console

   $ cd NAME
   $ source setup_stv.sh
   $ make

Sourcing the ``setup_stv.sh`` bash script is all that is necessary to
initialize the runtime environment again during a fresh login.

How to run
----------

The `xsec-analyzer <https://github.com/LiangLiu212/xsec_analyzer/tree/docs>`_ provides 
for major analysis tools:

1. `ProcessNTuples`
2. `univmake`
3. `Slice_Plots`
4. `Unfolder`


