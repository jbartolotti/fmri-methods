.. _gimmestart:

gimmefMRI
==============

``gimmefMRI`` provides an interface for running GIMME models with the ``gimme`` R package (Group Iterative Multiple Model Estimation) on fMRI timecourse data for functional connectivity analyses. 

.. _gimmesetup
Setup
--------------

Install the `gimmefMRI <https://github.com/jbartolotti/gimmefMRI>`_ package in R using the following two commands:

.. code-block:: console

    install.packages('devtools')
    devtools::install_github('jbartolotti/gimmefMRI')

Load the gimmefMRI library: 

.. code-block:: console

    library(gimmefMRI)

**NOTE:** On systems that use an outdated g++ compiler (including CentOS 7 or RHEL 7), one of the dependent packages (gridtext) will fail to install. You need to install an updated compiler and direct R to use it; see https://stackoverflow.com/questions/63962253/problem-compiling-the-%c2%b4gridtext%c2%b4-package-in-r/66811910#66811910


This package comes with built-in dummy data to test your installation. Use the following command: 

.. code-block:: console

    gimmefMRI(mode = 'demo')

This will create subfolders ``models`` and ``scripts`` in your current directory. ``scripts`` contains the file ``run_models.R`` which contains the R-code necessary to run the two pre-configured gimme models. ``models`` contains two subfolders, ``first_model`` and ``second_model``, each of which contains the input data, model output, and sample figures.

.. _gimmeusage:

Usage
------------

The core of the package is a data/configuration excel file that contains all timecourse data for a project, parameters for each model to be run, and parameters for each figure to generate. Running ``gimmefMRI()`` prompts you to select the configuration file, at which point input data is prepared, model code is written and executed, and figure code is written and executed. 

The function ``getTC()`` will generate a ``timecourses.csv`` file suitable for use as the data sheet in a ``gimme_config.xlsx`` file. Running ``getTC()`` will prompt you to select a ``get_timecourses.csv`` configuration file. This file contains rows for each subject and for each ROI. Subject rows provide paths to the preprocessed functional brain data, anatomical mask, and (optional) motion censoring timecourse, as well as where to save the single-roi timecourse files. ROI rows provide paths to each ROI mask. 

The output ``timecourses.csv`` file contains columns for each ROI, plus data columns including subject, time, group, condition, run, and censoring.

**NOTE:** AFNI functions must be installed and on the path before opening R in order to run. On the Synapse research server, use ``load afni`` to add it to the path. Alternatively, ``getTC()`` generates an ``extract_timecourses.sh`` file that can be run from the command line. This will create individual files for each combination of subject and ROI. In a later update, ``getTC()`` will allow you to generate the ``timecourses.csv`` file from these single-roi timecourse files directly.

To generate sample configuration files ``DemoGIMME.xlsx`` and ``get_timecourses.csv`` in the specified target directory, run the following. If ``writedir`` is not specified, the default is to save the sample files in the current directory. 

.. code-block:: console

    gimmefMRI_templates(writedir = TARGET_DIRECTORY)

.. _gimmeconfig:

Configuring GIMME.xlsx
~~~~~~~~~~~~~~

download `DemoGIMME.xlsx <https://github.com/jbartolotti/gimmefMRI/blob/main/inst/extdata/DemoGIMME.xlsx>`_

All data to be analyzed is located in the TIMECOURSES sheet. This contains a single column for each ROI or other predictor (e.g., task) of interest. Models may use all or a subset of these predictors. Additional columns specify Subject, Subgroup, Run, Condition, Slice Number, and Time. The Censor column can be used to exclude single rows from the model (1 = exclude). 

The CONTROL sheet specifies where data is stored, where results should be saved, and which parts of the analysis to run.

The MODELS sheet contains any number of columns, each one specifying a single GIMME model to run.

The LISTS sheet contains lists of nodes or subjects to include in a GIMME model, and is referred to within the MODELS configuration.

The ABBREVIATIONS sheet provides a mapping between long names and shortnames for network nodes. Longnames refer to column names in TIMECOURSES that are specified in the LISTS sheet. Shortnames are used in figures.

The FIGURES sheet contains any number of columns, each one specifying a single network figure to create for a specified model.

.. _gettcconfig:

Configuring get_timecourses.csv
~~~~~~~~~~~~~

*Under Construction*

.. _gimmerun:

Runtime Options
~~~~~~~~~~~~~~~~

.. code-block:: console

    gimmefMRI('load', 
      run = c(generate_models = TRUE, run_models = TRUE, generate_figures = TRUE, run_figures = TRUE),
      models = c('model1','model2')
      )

The first argument is the path to the ``GIMME.xlsx`` configuration file. Default is `load` to prompt the user for the file location interactively. `demo` runs using a built-in dataset.

Use the ``run`` option to specify steps to run or skip. If empty, values from the CONTROL sheet in GIMME.xlsx will be used. Run options specified in the command override values in the CONTROL sheet.

Use the `models` option to specify models to run for each step. Default: run all models listed in the MODELS sheet.
