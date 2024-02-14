Preprocessing fMRI data with fMRIPrep
================

.. _setup:

Setup
--------------

Install Singularity and Anaconda

Build fmriprep singularity image in ``/usr/local/singularity``

Create a new conda environment for bidskit and fmriprep

[Obtain freesurfer license](https://surfer.nmr.mgh.harvard.edu/fswiki/License) and place in folder for mounting



.. _usage:

Usage
------------

Ensure that the dataset is BIDS formatted for processing with fMRIPrep

.. code-block:: console

   bidskit

Singularity will by default make environment variables available to the container, which can expose unintended software to fMRIPrep (e.g., the user's FSL version instead of the container's built-in version).
Use the ``--cleanenv`` flag to avoid passing environment variables.

However, you do need to pass an environment variable containing the path to the freesurfer license file. Do this by creating a ``APPTAINERENV_FS_LICENSE`` variable, which singularity will pass to the container as ``FS_LICENSE``.
After specifying the path to the license file, e.g. ``/opt/fslicense/license.txt``, you must also bind the folder on the host system to this new path. The option ``--bind /usr/local/fslicense:/opt/fslicense`` will map the contents of ``/usr/local/fslicense`` on the host (where the file actually resides) to the path ``/opt/fslicense`` within the container (where fMRIPrep will look for it).

fMRIPrep makes extensive use of a temporary directory, ``/tmp`` for intermediate files. ``/tmp`` is automatically bound from the host system to the container, so fMRIPrep will use ``/tmp`` on the host for its working directory. Depending on system configuration, ``/tmp`` may not have sufficient disk space alloted, causing the processing job to halt and error before completing. To remedy this, map a different location on the host system that is unrestricted in size to ``/tmp``, e.g. ``$HOME/tmp:/tmp``. fMRIPrep will then put its working directory in ``$HOME/tmp/work`` on the host system.

*NB: Clear this custom tmp directory regularly when you no longer need the intermediate working files.*

If fMRIPrep aborts partway through, you can rerun it from where it stopped by using the ``-w`` option and passing the work directory from the failed attempt. Note, this refers to the work directory within the container, i.e. ``/tmp/work``, not the path to the work directory on the host sytem.

**Multi-echo preprocessing**

You must add the option ``--me-output-echos`` for fMRIPrep to write processed files for each echo that you can optimally combine with tedana.

The syntax for running fMRIPrep in a singularity container is generally:

.. code-block:: console
   singularity run [singularity options] /path/to/container bids_dir output_dir participant [fMRIPrep options]

bids_dir is the root folder of a BIDS valid dataset (sub-XXXXX folders should be found at the top level in this folder).

output_dir is where the preprocessed files and reports will be saved

.. code-block:: console

   conda activate fmriprep-env

   export APPTAINERENV_FS_LICENSE=/opt/fslicense/license.txt
   singularity run --cleanenv --bind /usr/local/fslicense:/opt/fslicense,$HOME/tmp:/tmp /usr/local/singularity/nipreps_fmriprep_23.2.0-2024-01-10-63081a7fe2b8.simg \
       $HOME/path/to/bidsdata $HOME/path/to/bidsdata/derivatives \
       participant \
       --me-output-echos \
       -w /tmp/work