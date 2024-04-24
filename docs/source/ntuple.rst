Ntuple post-processing
======================

``ProcessNTuples`` in ``$STV_ANALYSIS_DIR/NTupleProcessing`` converts the ROOT ntuple 
files produced by PeLEE team's `searchingfornues <https://github.com/ubneutrinos/searchingfornues>`_ framework
into a new, post-processed ntuple file. The workflow of ``ProcessNTuples`` includes three steps

- Read ntuples from files produced by PeLEE team's
- Select, classify (for MC) and compute new observables for each event
- Save the survived events into a new ROOT file.


Input ntuples
-------------

Ntuple files from PeLEE include two ``TTree``: ``nuselection/NeutrinoSelectionFilter`` and ``nuselection/SubRun``. 

1. ``nuselection/NeutrinoSelectionFilter``

2. ``nuselection/SubRun`` contains the informations of proton on target (POT) for the current sub run

====== ====== ===============================================
Branch Type   Description
------ ------ -----------------------------------------------
run    int    Run number
subRun int    subRun number
pot    float  The total amount of POT for the current sub run
====== ====== ===============================================

``TChain`` is used to manipulate multiple files.






