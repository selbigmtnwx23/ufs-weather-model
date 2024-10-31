.. |nbsp| unicode:: 0xA0 
   :trim:

.. role:: raw-html(raw)
    :format: html

.. _hsd:

********************************************
Hierarchical System Development (HSD) Cases
********************************************






The UFS Weather Model (WM) can be run in any of several configurations, from a single-component atmospheric 
model to a fully coupled model with multiple earth system components (e.g., atmosphere, ocean, sea-ice, land, and 
mediator). This chapter documents a few of the cases designed to support hierarchical system development (HSD) within the UFS. 
For a full list of supported WM configurations, view the `rt.conf <https://github.com/ufs-community/ufs-weather-model/blob/develop/tests/rt.conf>`__ file.

.. attention::

   This chapter is a work in progress. There are a multitude of options for configuring the UFS WM, 
   and this chapter merely details a few supported configurations. It will be expanded over time
   to include a wide variety of idealized test cases for use in research and testing. 

.. _ufs-test:

================
``ufs_test.sh``
================

This section will include details on how to run idealized cases using the ``ufs-test.sh`` script.

Clone the Repository
--------------------

To start, clone the branch containing the test cases:

.. code-block:: console

   git clone --recursive -b feature/enable_test_cases https://github.com/natalie-perlin/ufs-weather-model.git

After cloning, set up the environment:

.. code-block:: console

   cd ufs-weather-model
   export UFS_WM=$PWD         # This variable is for convenience
   cd tests-dev

The tests are configured to be run on NOAA Tier1 platforms, and the configuration files for each platform are located at:

.. code-block:: console

   ${UFS_WM}/tests-dev/machine_config/machine_<PLATFORM>.config

This file loads the necessary Python and Rocoto modules for each platform.

Baseline Configuration
----------------------

Another configuration file, ``${UFS_WM}/tests-dev/baseline_setup.yaml``, contains details for staged input data location, user-specific output directories, and batch job scheduling. Verify the following variables in this file:

- ``dprefix``: Ensure the directory exists and you have write permissions.
- ``STMP``: Directory for baseline test outputs.
- ``PTMP``: Directory for runtime files.

Running Tests
-------------

Launch tests from the ``${UFS_WM}/tests-dev`` directory with the following command:

.. code-block:: console

   ./ufs_test.sh -a <ACCOUNT> [-s] [-c] -k -r -n "<CASE_NAME> <COMPILER>"

where:

- ``<ACCOUNT>``: Account/project number for batch jobs.
- ``<CASE_NAME>``: Name of the test case (e.g., ``2020_CAPE`` or ``baroclinic_wave``).
- ``<COMPILER>``: Compiler used for the tests (``intel`` or ``gnu``).

**Comand-line Options:**

- ``-s``: Sync scripts (only required on the first run, it syncs scripts from ``./ufs-wm/tests`` to ``./ufs-wm/tests-dev``.).
- ``-c``: Create a baseline (necessary until baselines are staged).
- ``-k``: Keep runtime directories after test completion.
- ``-r``: Use Rocoto workflow manager.
- ``-n``: Run a single test case.

Example Commands
----------------

To run the ``2020_CAPE`` test case with the ``intel`` compiler on ``Hera``, ``Orion``, or ``Gaea``:

.. code-block:: console

   ./ufs_test.sh -a epic -s -c -k -r -n "2020_CAPE intel"

For the ``baroclinic_wave`` test case, which takes longer:

.. code-block:: console

   ./ufs_test.sh -a epic -s -c -k -r -n "baroclinic_wave intel"

Running Multiple Cases
----------------------

To run multiple cases at once, copy ``test_cases.yaml`` from the test cases directory, then use the ``-l`` argument:

.. code-block:: console

   cp ${UFS_WM}/tests-dev/test_cases/test_cases.yaml ${UFS_WM}/tests-dev/
   ./ufs_test.sh -a epic -s -c -k -r -l test_cases.yaml

Accessing Run and Output Files
------------------------------

Compilation and model run directories can be accessed in the local repository via the ``run_dir`` softlink, which points to the actual ``FV3_RT`` directory. Each test generates ``atm*.nc`` and ``sfc*.nc`` files at specified forecast hour intervals. To monitor progress:

.. code-block:: console

   tail -f <file>  

Once the tests run successfully with the ``-c`` option (baseline created), future tests can compare results with the new baseline using ``-m`` instead of ``-c``.

.. note::

   After the initial run of ``ufs_test.sh`` with the ``-s`` option, you do not need to use it again. Once the baseline is created, you can also use the ``-m`` option to compare with the new baseline if additional testing is done within the same local clone.

For further test management, you may save the test directory location in an environment variable for convenience:

.. code-block:: console

   export UFS_WM_TEST=/path/to/expt_dirs/ufs_test

.. _cape-2020:

====================
2020 July CAPE Case
====================

The July 2020 CAPE case illustrates one of the shortcomings of the Global Forecast System (GFS) v16: low Convective Available Potential Energy (CAPE) predictions during summertime. The NOAA Environmental Modeling Center (EMC) Model Evaluation Group (MEG) identified this concern, and .

The case runs are initialized at 00z Jul 23, 2020 with a 24 hour forecast length. The corresponding namelist options that need to be changed are listed below. The app uses ./xmlchange to change the runtime settings. The settings that need to be modified to set up the start date, start time, and run time are listed below.

Initial condition (IC) files are created from GFS operational dataset in NEMSIO format. The GFS analysis dataset is used as ‘truth’ to compare with simulated synoptic dynamic fields. The CAPE field is evaluated based on Rapid Refresh (RAP) analysis dataset and atmospheric sounding.

Both MRW App v1.0 and GFS.v16.0.10 simulate a lower value of CAPE compared with RAP_ANL and sounding observation in this summertime case study. Further investigations (MEG 2021) show that this is related to the drier soil layers in GFS initial conditions. The SRW_RRFSv1alpha also underestimates the CAPE. (:cite:t:`SunEtAl2024`)

References

NOAA Environmental Modeling Center Model Evaluation Group (MEG) (2021). [Link]

Sun X., D. Heinzeller, L. Bernardet, L. Pan, W. Li, D. Turner, and J. Brown. 2024: A Case Study Investigating the Low Summertime CAPE Behavior in the Global Forecast System. Weather and Forecasting. 

https://doi.org/10.1175/WAF-D-22-0208.1

https://journals.ametsoc.org/view/journals/wefo/39/1/WAF-D-22-0208.1.xml
Last name and initials of author(s) (if nine or more, the first author is followed by "and Coauthors"), year of publication, title of paper, title of journal (italicized),* volume of journal (bolded), issue or citation number (only if required for identification), page range, and DOI (if available).

export dprefix="/scratch2/NAGAPE"
STMP="${dprefix}/stmp4"
PTMP="${dprefix}/stmp2"

.. _baroclinicwave:

============================
Baroclinic Instability Case
============================

The baroclinic wave test case evaluates the performance of dry dynamical cores in atmospheric models, focusing on the idealized growth of a northern hemisphere wave. The initial zonal state is a quasi-realistic steady solution to the adiabatic, inviscid primitive equations, specified analytically. The test begins with an assessment of whether the models maintain this steady state, followed by a perturbation that induces baroclinic wave growth over several days.

Four dynamical cores with varying resolutions are tested: NASA/NCAR’s Finite Volume package, NCAR’s spectral transform Eulerian and semi-Lagrangian cores in CAM3, and the German Weather Service’s GME model. These hydrostatic cores, which span a range of numerical methods, offer independent high-resolution reference solutions. The study analyzes each model's convergence characteristics and explores the uncertainty of the high-resolution results.

Refernces 

