Ntuple post-processing
----------------------

``ProcessNTuples`` in ``$STV_ANALYSIS_DIR/NTupleProcessing`` converts the ROOT ntuple 
files produced by PeLEE team's `searchingfornues <https://github.com/ubneutrinos/searchingfornues>`_ framework
into a new, post-processed ntuple file. The workflow of ``ProcessNTuples`` includes three steps

- Read ntuples from files produced by PeLEE team's
- Select, classify (for MC) and compute new observables for each event
- Save the survived events into a new ROOT file.

``TChain`` is used to manipulate multiple files.

Input ntuples
~~~~~~~~~~~~~

Ntuple files from PeLEE include two ``TTree``: ``nuselection/NeutrinoSelectionFilter`` and ``nuselection/SubRun``. 

1. ``nuselection/NeutrinoSelectionFilter`` contains more than 700 branchs for each event. These branchs include the information of reconstructed showers, tracks, cosmic rays, such as the MC truth, particle identification, reconstructed four momentum, position of vertex, etc. ``Branches.h`` in ``$STV_ANALYSIS_DIR/Utils/Includes`` provide helper function to read the information from the event TTree. The objects ``AnalysisEvent`` will hold the information for the procedures in next step. Here is a slice from ``$STV_ANALYSIS_DIR/Utils/Includes/Branches.h``. Users can modify this file to read branchs for their specific purposes.

.. code-block:: cpp
    void set_event_branch_addresses(TTree& etree, AnalysisEvent& ev){

	    // ......

	    // Reconstructed neutrino vertex position (with corrections for
	    // space charge applied)
	    SetBranchAddress(etree, "reco_nu_vtx_sce_x", &ev.nu_vx_ );
	    SetBranchAddress(etree, "reco_nu_vtx_sce_y", &ev.nu_vy_ );
	    SetBranchAddress(etree, "reco_nu_vtx_sce_z", &ev.nu_vz_ );

	    // ......
    }

2. ``nuselection/SubRun`` contains the informations of proton on target (POT) for the current sub run

====== ====== ===============================================
Branch Type   Description
====== ====== ===============================================
run    int    Run number
subRun int    subRun number
pot    float  The total amount of POT for the current sub run
====== ====== ===============================================


