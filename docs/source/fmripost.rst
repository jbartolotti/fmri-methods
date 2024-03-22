Postprocessing fMRI data in BIDS format
================

.. _setup:

Setup
--------------
**IMPORTANT:** These installation steps should already be done for you if using an HBIC managed environment, e.g. Synapse.

Install Singularity

Build xcp-d image in ``/usr/local/singularity`` with the appropriate version number (see version list `here <https://xcp-d.readthedocs.io/en/latest/changes.html>`_ )

.. code-block:: console

   singularity build xcp_d-<version>.simg docker://pennlinc/xcp_d:<version>

XCP-D
----------------------

`XCP-D <https://xcp-d.readthedocs.io/en/latest/index.html>`_ is an fMRI post-processing and noise regression pipeline designed for **resting-state functional connectivity** data. XCP-D takes as input the output of fMRIPrep, as well as the outputs of several large imaging databases.

.. code-block:: console

   export APPTAINERENV_FS_LICENSE=/opt/fslicense/license.txt
   singularity run --cleanenv --bind /usr/local/fslicense:/opt/fslicense,$HOME/tmp:/tmp \
      /usr/local/singularity/xcp_d-0.6.4.simg \
      $HOME/path/to/fmriprepdata $HOME/path/to/fmriprepdata/derivatives/xcp_d \
      --nthreads 8 -w /tmp/xcpd --fs-license-file /usr/local/fslicense \
      --nuisance-regressors 36P  --smoothing 6 \
      --participant-label TD001 \
      --task-id rest

