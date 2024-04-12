Analysis workflow
=====

The entire :math:`{\rm CC}0\pi Np` analysis is based on the ROOT ntuple files produced by the
PeLEE team's `searchingfornues <https://github.com/ubneutrinos/searchingfornues>`_ framework. A list of pre-made
PeLEE ntuple files for beam data, cosmic data, central-value Monte Carlo (MC),
and detector variation MC samples is maintained in
`files\_to\_process.txt`, and the samples themselves are described in
the :math:`{\rm CC}0\pi Np` analysis note `MCC9 Double-differential CCNp cross sections <https://microboone-docdb.fnal.gov/cgi-bin/sso/ShowDocument?docid=35518>`_. Any processing of data or MC samples into new PeLEE ntuple files must be handled upstream of the
`stv-analysis-new <https://github.com/LiangLiu212/xsec_analyzer/tree/docs>`_ tools.

Assuming that the user has all needed PeLEE ntuple files already processed, the
major steps for performing a measurement are outlined in the following
subsections.
