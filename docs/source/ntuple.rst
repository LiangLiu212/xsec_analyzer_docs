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

1. ``nuselection/NeutrinoSelectionFilter`` contains more than 700 branchs for each event. These branchs include the information of reconstructed showers, tracks, cosmic rays, such as the MC truth, particle identification, reconstructed four momentum, position of vertex, etc. ``Branches.h`` in ``$STV_ANALYSIS_DIR/Utils/Includes`` provide helper function to read the information from the event TTree. The object ``AnalysisEvent`` will hold the information for the procedures in next step. Here is a slice from ``$STV_ANALYSIS_DIR/Utils/Includes/Branches.h``. Users can modify this file to read branchs for their specific purposes.

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




============================ ============================== ======= ===========
Branch			     AnalysisEvent		    Type    Description
============================ ============================== ======= ===========
slpdg                        ev.nu_pdg_                     INT     adsas
nslice                       ev.nslice_                     INT     
topological_score            ev.topological_score_          INT
CosmicIP                     ev.cosmic_impact_parameter_    INT
CosmicIPAll3D                ev.CosmicIPAll3D_              INT
reco_nu_vtx_sce_x            ev.nu_vx_                      INT
reco_nu_vtx_sce_y            ev.nu_vy_                      INT
reco_nu_vtx_sce_z            ev.nu_vz_                      INT
n_pfps                       ev.num_pf_particles_           INT
n_tracks                     ev.num_tracks_                 INT
n_showers                    ev.num_showers_                INT
nu_pdg                       ev.mc_nu_pdg_                  INT
true_nu_vtx_x                ev.mc_nu_vx_                   INT
true_nu_vtx_y                ev.mc_nu_vy_                   INT
true_nu_vtx_z                ev.mc_nu_vz_                   INT
nu_e                         ev.mc_nu_energy_               INT
ccnc                         ev.mc_nu_ccnc_                 INT
interaction                  ev.mc_nu_interaction_type_     INT
true_nu_vtx_sce_x            ev.mc_nu_sce_vx_               INT
true_nu_vtx_sce_y            ev.mc_nu_sce_vy_               INT
true_nu_vtx_sce_z            ev.mc_nu_sce_vz_               INT
weightSpline                 ev.spline_weight_              INT
weightTune                   ev.tuned_cv_weight_            INT
nu_completeness_from_pfp     ev.nu_completeness_from_pfp_   INT
nu_purity_from_pfp           ev.nu_purity_from_pfp_         INT
pfp_generation_v             ev.pfp_generation_             INT
pfp_trk_daughters_v          ev.pfp_trk_daughters_count_    INT
pfp_shr_daughters_v          ev.pfp_shr_daughters_count_    INT
trk_score_v                  ev.pfp_track_score_            INT
pfpdg                        ev.pfp_reco_pdg_               INT
pfnhits                      ev.pfp_hits_                   INT
pfnplanehits_U               ev.pfp_hitsU_                  INT
pfnplanehits_V               ev.pfp_hitsV_                  INT
pfnplanehits_Y               ev.pfp_hitsY_                  INT
backtracked_pdg              ev.pfp_true_pdg_               INT
backtracked_e                ev.pfp_true_E_                 INT
backtracked_px               ev.pfp_true_px_                INT
backtracked_py               ev.pfp_true_py_                INT
backtracked_pz               ev.pfp_true_pz_                INT
shr_pfp_id_v                 ev.shower_pfp_id_              INT
shr_start_x_v                ev.shower_startx_              INT
shr_start_y_v                ev.shower_starty_              INT
shr_start_z_v                ev.shower_startz_              INT
shr_dist_v                   ev.shower_start_distance_      INT
trk_pfp_id_v                 ev.track_pfp_id_               INT
trk_len_v                    ev.track_length_               INT
trk_sce_start_x_v            ev.track_startx_               INT
trk_sce_start_y_v            ev.track_starty_               INT
trk_sce_start_z_v            ev.track_startz_               INT
trk_distance_v               ev.track_start_distance_       INT
trk_sce_end_x_v              ev.track_endx_                 INT
trk_sce_end_y_v              ev.track_endy_                 INT
trk_sce_end_z_v              ev.track_endz_                 INT
trk_dir_x_v                  ev.track_dirx_                 INT
trk_dir_y_v                  ev.track_diry_                 INT
trk_dir_z_v                  ev.track_dirz_                 INT
trk_theta_v                  ev.track_theta_                INT
trk_phi_v                    ev.track_phi_                  INT
trk_energy_proton_v          ev.track_kinetic_energy_p_     INT
trk_range_muon_mom_v         ev.track_range_mom_mu_         INT
trk_mcs_muon_mom_v           ev.track_mcs_mom_mu_           INT
trk_pid_chipr_v              ev.track_chi2_proton_          INT
trk_llr_pid_v                ev.track_llr_pid_              INT
trk_llr_pid_u_v              ev.track_llr_pid_U_            INT
trk_llr_pid_v_v              ev.track_llr_pid_V_            INT
trk_llr_pid_y_v              ev.track_llr_pid_Y_            INT
trk_llr_pid_score_v          ev.track_llr_pid_score_        INT
mc_pdg                       ev.mc_nu_daughter_pdg_         INT
mc_E                         ev.mc_nu_daughter_energy_      INT
mc_px                        ev.mc_nu_daughter_px_          INT
mc_py                        ev.mc_nu_daughter_py_          INT
mc_pz                        ev.mc_nu_daughter_pz_          INT
weights                      ev.mc_weights_map_             INT
============================ ============================== ======= ===========


2. ``nuselection/SubRun`` contains the informations of proton on target (POT) for the current sub run

====== ====== ===============================================
Branch Type   Description
====== ====== ===============================================
run    int    Run number
subRun int    subRun number
pot    float  The total amount of POT for the current sub run
====== ====== ===============================================



Selection
~~~~~~~~~


