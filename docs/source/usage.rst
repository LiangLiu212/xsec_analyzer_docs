Quick Start
===========

.. _installation:

Installation
------------

The `xsec-analyzer <https://github.com/LiangLiu212/xsec_analyzer/tree/docs>`_ framework is designed to be installed and run on te uboonegpvms.
In principle it can be used elsewhere if you have access to the 'PeLEE' ntuples. The only external dependency is ROOT.
To install the code, navigate to a writeable area in your


.. code-block:: console

    $ cd your_workarea
    $ git clone https://github.com/uboone/xsec_analyzer.git
    $ git checkout –t origin/tutorial-umn
    $ source setup_xsec_analyzer.sh
    $ make
   

The setup script will attempt to set up ROOT. It will auto-detect whether you're running SL7 or AL9.
If you're running SL7, the setup script will set up ROOT via uboonecode. If you're running
AL9, ROOT will be set up by spack.

The setup script will set the path to source code in the environment variable ``XSEC_ANALYZER_DIR``. This is
used by the framework to find certain files, add ``${XSEC_ANALYZER_DIR}/bin`` into the environment variable ``PATH``,
add ``${XSEC_ANALYZER_DIR}/lib`` into the environment variable ``LD_LIBRARY_PATH``

On a fresh login, just navigate to the same folder and source the ``setup_xsec_analyzer.sh`` script.


How to run
----------

The `xsec-analyzer <https://github.com/LiangLiu212/xsec_analyzer/tree/docs>`_ provides 
six major analysis tools. They are executables in ``${XSEC_ANALYZER_DIR}/bin``:

.. code-block:: console

    $ ls ${XSEC_ANALYZER_DIR}/bin
    BinScheme  ProcessNTuples   SlicePlots  StandaloneUnfold  Unfolder  univmake

Here, I use ``CC1mu1p0pi`` channel as an example to show the workflow of xsec-analyzer.

- STEP 1. ``ProcessNTuples``

    .. code-block:: console

        $ ProcessNTuples INPUT_PELEE_NTUPLE_FILE CC1mu1p0pi OUTPUT_FILE

    - ``INPUT_PELEE_NTUPLE_FILE`` should be a PeLEE ntuples, e.g. `PeLEE Samples 2023 <https://docs.google.com/spreadsheets/d/1dX-W4DGTHeZbJLt2HvwXS4QDNeEwYKveHHSCkVrJcSU/edit?gid=0#gid=0>`_

    - The second input argument is the name of selection algorithm. The ``CC1mu1p0pi`` is implement in

        - $XSEC_ANALYZER_DIR/include/XSecAnalyzer/Selections/CC1mu1p0pi.hh
        - $XSEC_ANALYZER_DIR/src/selections/CC1mu1p0pi.cxx

    - ``OUTPUT_FILE`` is the post-processed ntuple. It includes a TParameter called ``summed_pot`` and a TTree called ``stv_tree``

        For MC samples, the ``summed_pot`` gives the simulated POT needed to scale to data. For real data, it is always zero. The POT and trigger counts for real data
        files are stored elsewhere (more on that later). This scaling is handled automatically by later parts of the framework, so
        no need to worry much about this item.

        Many branches in ``stv_tree`` are copied over directly from the PeLEE ntuples, some
        are new and analysis-specific.
        Name is a hold-over from a much older incarnation of the code

- STEP 2. ``BinScheme``




3. `univmake`
4. `Slice_Plots`
5. `Unfolder`
6. `StandaloneUnfold`


Firstly, you need to reprocess ntuples:

.. code-block:: console

	$ ./Scripts/ReprocessNTuples.sh OUTPUT_DIRECTORY NTUPLE_LIST_FILE

``NTUPLE_LIST_FILE`` is a plain text which collects ntuple files from `PeLEE <https://github.com/ubneutrinos/searchingfornues>`_. 
The location of ntuples can be found in `spreadsheet <https://docs.google.com/spreadsheets/d/1dX-W4DGTHeZbJLt2HvwXS4QDNeEwYKveHHSCkVrJcSU/edit?gid=0#gid=0>`_, as well as their Trigger counts, POT exposures and scaling factor. 
``OUTPUT_DIRECTORY`` is the directory of the outputs. 

Secondly, you can make universes by:

.. code-block:: console

	$ ./Scripts/UniverseMaker.sh FPM_CONFIG SEL_CONFIG OUTPUTFILE

``FPM_CONFIG`` includes the output from the last step. 
