Ntuple post-processing
----------------------

What does ``ProcessNTuples`` do?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``ProcessNTuples`` converts the ROOT ntuple
files produced by PeLEE team's `searchingfornues <https://github.com/ubneutrinos/searchingfornues>`_ framework
into a new, post-processed ntuple file.

- Define signal and background
- Apply selection criteria (using a bool to label the survived events instead of applying a filter)
- Compute true observables
- Compute reconstruction observables
- Categorize event

1. SelectionBase

    `SelectionBase <https://github.com/uboone/xsec_analyzer/blob/main/include/XSecAnalyzer/Selections/SelectionBase.hh>`_ is the base class of all selection classes in analyzer framework. It
    provides the common interface for operations such as setup input branches, define signal, selection, define output branches ...

    .. code-block:: cpp

          virtual bool selection( AnalysisEvent* event ) = 0;
          virtual int categorize_event( AnalysisEvent* event ) = 0;
          virtual void compute_reco_observables( AnalysisEvent* event ) = 0;
          virtual void compute_true_observables( AnalysisEvent* event ) = 0;
          virtual void define_output_branches() = 0;
          virtual bool define_signal( AnalysisEvent* event ) = 0;
          virtual void define_constants() = 0;
          virtual void define_category_map() = 0;
          virtual void reset() = 0;
          virtual void define_additional_input_branches(TTree& etree) = 0;


2. SelectionFactory

    `SelectionFactory <https://github.com/uboone/xsec_analyzer/blob/main/include/XSecAnalyzer/Selections/SelectionFactory.hh>`_
    implements a factory design pattern to dynamically create objects of different selection classes
    (CC1mu1p0pi, CC1mu2p0pi, etc.) based on a string input. It provides a centralized way to manage the creation of
    these objects while ensuring that unsupported requests are handled with an error message and exception.

    Users need to put their own selection classes into ``SelectionFactory::CreateSelection``

    .. code-block:: cpp

        SelectionBase* SelectionFactory::CreateSelection(
          const std::string& selection_name )
        {
          SelectionBase* sel;
          if ( selection_name == "CC1mu1p0pi" ) {
            sel = new CC1mu1p0pi;
          }
          else if ( selection_name == "CC1mu2p0pi" ) {
            sel = new CC1mu2p0pi;
          }
          else if ( selection_name == "CC1muNp0pi" ) {
            sel = new CC1muNp0pi;
          }
          else if ( selection_name == "Dummy" ) {
            sel = new DummySelection;
          }
          else {
            std::cerr << "Selection name requested: " << selection_name
              << " is not implemented in " << __FILE__ << '\n';
            throw;
          }

Input ntuples
~~~~~~~~~~~~~

Ntuple files from PeLEE include two ``TTree``: ``nuselection/NeutrinoSelectionFilter`` and ``nuselection/SubRun``. 

1. ``nuselection/NeutrinoSelectionFilter`` contains more than 700 branchs for each event. These branchs include the information of reconstructed showers, tracks, cosmic rays, such as the MC truth, particle identification, reconstructed four momentum, position of vertex, etc.
   `Branches.hh <https://github.com/uboone/xsec_analyzer/blob/main/include/XSecAnalyzer/Branches.hh>`_ provide helper function to read the information from the event TTree. The object `AnalysisEvent <https://github.com/uboone/xsec_analyzer/blob/main/include/XSecAnalyzer/AnalysisEvent.hh>`_ will hold the information for the procedures in next step.

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
slpdg                        ev.nu_pdg_                     INT     Reco PDG code of primary PFParticle in slice (i.e., the neutrino candidate)
nslice                       ev.nslice_                     INT     Number of neutrino slices identified by the SliceID. Allowed values are zero or one.
topological_score            ev.topological_score_          FLOAT     A topological score which assesses to what extent the slice looks like a neutrino interaction in the TPC
CosmicIP                     ev.cosmic_impact_parameter_    FLOAT     3D distance of shower start from closest spacepoint of primary muon (i.e. cosmic)
reco_nu_vtx_sce_x            ev.nu_vx_                      FLOAT     x component of reconstructed neutrino vertex position (with corrections for space charge applied)
reco_nu_vtx_sce_y            ev.nu_vy_                      FLOAT     y component of reconstructed neutrino vertex position (with corrections for space charge applied)
reco_nu_vtx_sce_z            ev.nu_vz_                      FLOAT     z component of reconstructed neutrino vertex position (with corrections for space charge applied)
n_pfps                       ev.num_pf_particles_           INT     Number of pandora final particles
n_tracks                     ev.num_tracks_                 INT     Number of tracks in pandora final particles
n_showers                    ev.num_showers_                INT     Number of showers in pandora final particles
nu_pdg                       ev.mc_nu_pdg_                  INT     PDG id of nu (MC truth)
true_nu_vtx_x                ev.mc_nu_vx_                   FLOAT     x component of truth neutrino vertex coordinates
true_nu_vtx_y                ev.mc_nu_vy_                   FLOAT     y component of truth neutrino vertex coordinates
true_nu_vtx_z                ev.mc_nu_vz_                   FLOAT     z component of truth neutrino vertex coordinates
nu_e                         ev.mc_nu_energy_               FLOAT     Truth neutrino energy
ccnc                         ev.mc_nu_ccnc_                 int     Whether the event is CC (0) or NC (1)     
interaction                  ev.mc_nu_interaction_type_     INT     Interaction code from GENIE
true_nu_vtx_sce_x            ev.mc_nu_sce_vx_               INT     x component of truth neutrino vertex position (with corrections for space charge applied)
true_nu_vtx_sce_y            ev.mc_nu_sce_vy_               INT     y component of truth neutrino vertex position (with corrections for space charge applied)
true_nu_vtx_sce_z            ev.mc_nu_sce_vz_               INT     z component of truth neutrino vertex position (with corrections for space charge applied)
weightSpline                 ev.spline_weight_              INT     GENIE weights
weightTune                   ev.tuned_cv_weight_            INT     GENIE weights
nu_completeness_from_pfp     ev.nu_completeness_from_pfp_   INT     Completeness of the backtracked hits in the neutrino slice
nu_purity_from_pfp           ev.nu_purity_from_pfp_         INT     Purity of the backtracked hits in the neutrino slice
pfp_generation_v             ev.pfp_generation_             INT     generation, 1 is primary
pfp_trk_daughters_v          ev.pfp_trk_daughters_count_    INT     number of track daughters
pfp_shr_daughters_v          ev.pfp_shr_daughters_count_    INT     number of shower daughters
trk_score_v                  ev.pfp_track_score_            INT     
pfpdg                        ev.pfp_reco_pdg_               INT     PDG code of pfp in slice
pfnhits                      ev.pfp_hits_                   INT     number of hits in pfp
pfnplanehits_U               ev.pfp_hitsU_                  INT     number of hits in pfp plane U
pfnplanehits_V               ev.pfp_hitsV_                  INT     number of hits in pfp plane V
pfnplanehits_Y               ev.pfp_hitsY_                  INT     number of hits in pfp plane Y
backtracked_pdg              ev.pfp_true_pdg_               INT     PDG code of backtracked particle
backtracked_e                ev.pfp_true_E_                 INT     energy of backtracked particle
backtracked_px               ev.pfp_true_px_                INT     px of backtracked particle
backtracked_py               ev.pfp_true_py_                INT     py of backtracked particle
backtracked_pz               ev.pfp_true_pz_                INT     pz of backtracked particle
shr_pfp_id_v                 ev.shower_pfp_id_              INT     Shower properties
shr_start_x_v                ev.shower_startx_              INT     Shower properties
shr_start_y_v                ev.shower_starty_              INT     Shower properties
shr_start_z_v                ev.shower_startz_              INT     Shower properties
shr_dist_v                   ev.shower_start_distance_      INT     Shower properties
trk_pfp_id_v                 ev.track_pfp_id_               INT     Track properties
trk_len_v                    ev.track_length_               INT     Track properties
trk_sce_start_x_v            ev.track_startx_               INT     Track properties
trk_sce_start_y_v            ev.track_starty_               INT     Track properties
trk_sce_start_z_v            ev.track_startz_               INT     Track properties
trk_distance_v               ev.track_start_distance_       INT     Track properties
trk_sce_end_x_v              ev.track_endx_                 INT     Track properties
trk_sce_end_y_v              ev.track_endy_                 INT     Track properties
trk_sce_end_z_v              ev.track_endz_                 INT     Track properties
trk_dir_x_v                  ev.track_dirx_                 INT     Track properties
trk_dir_y_v                  ev.track_diry_                 INT     Track properties 
trk_dir_z_v                  ev.track_dirz_                 INT     Track properties
trk_theta_v                  ev.track_theta_                INT     Track properties
trk_phi_v                    ev.track_phi_                  INT     Track properties
trk_energy_proton_v          ev.track_kinetic_energy_p_     INT     Track properties
trk_range_muon_mom_v         ev.track_range_mom_mu_         INT     Track properties
trk_mcs_muon_mom_v           ev.track_mcs_mom_mu_           INT     Track properties
trk_pid_chipr_v              ev.track_chi2_proton_          INT     Track properties
trk_llr_pid_v                ev.track_llr_pid_              INT     Track properties
trk_llr_pid_u_v              ev.track_llr_pid_U_            INT     Track properties
trk_llr_pid_v_v              ev.track_llr_pid_V_            INT     Track properties
trk_llr_pid_y_v              ev.track_llr_pid_Y_            INT     Track properties
trk_llr_pid_score_v          ev.track_llr_pid_score_        INT     Track properties
mc_pdg                       ev.mc_nu_daughter_pdg_         INT     MC truth information for the final-state primary particles
mc_E                         ev.mc_nu_daughter_energy_      INT     MC truth information for the final-state primary particles
mc_px                        ev.mc_nu_daughter_px_          INT     MC truth information for the final-state primary particles
mc_py                        ev.mc_nu_daughter_py_          INT     MC truth information for the final-state primary particles
mc_pz                        ev.mc_nu_daughter_pz_          INT     MC truth information for the final-state primary particles
weights                      ev.mc_weights_map_             INT     General systematic weights
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



- Event Category

    Enum used to label event categories of interest for analysis plots in analyses.
    The enum is defined in header, e.g. `EventCategoriesXp.hh <https://github.com/uboone/xsec_analyzer/blob/main/include/XSecAnalyzer/Selections/EventCategoriesXp.hh>`_
    A map that associates specific event categories with a descriptive label and a color code for visualization purposes is defined
    in `EventCategoriesXp.cxx <https://github.com/uboone/xsec_analyzer/blob/main/src/selections/EventCategoriesXp.cxx>`_

    To use your event category

    .. code-block:: cpp

        void CC1muNp0pi::define_category_map() {
          // Use the shared category map for 1p/2p/Np/Xp
          categ_map_ = CC1muXp_MAP;
        }

    * Interaction codes and the corresponding processes

==== =================================
Code Process
==== =================================
0    NULL
1    QES (QuasiElastic)      
2    1Kaon (Single Kaon)    
3    DIS (Deep Inelastic)      
4    RES (Resonant)      
5    COH (Coherent Production)
6    DFR (Diffractive)      
7    NuEEL (Nu Electron Elastic)    
8    IMD (Inverse Mu Decay)     
9    AMNuGamma 
10   MEC (Meson Exchange)
11   CEvNS (Coherent Elastic)
12   IBD (Inverse Beta Decay)
13   GLR (Glashow Resonance)
14   IMDAnh (IMD Annihilation)
15   PhotonCOH (Photon Coherent)
16   PhotonRES (Photon Resonance)
101  DMEL (Dark Matter Elastic)
102  DMDIS (Dark Matter Deep Inelastic)
103  DME (Dark Matter Electron)
104  Norm      
-100 Uknown to GENIE
==== =================================


- MircoBooNE tune

    ``ev.spline_weight_`` and ``ev.tuned_cv_weight_`` are