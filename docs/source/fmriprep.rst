Preprocessing fMRI data with fMRIPrep
================

.. _setup:

Setup
--------------

Install Singularity and Anaconda

Build fmriprep singularity image in ``/usr/local/singularity``

Create a new conda environment for bidskit and fmriprep

Obtain freesurfer license and place in folder for mounting



.. _usage:

Usage
------------

Ensure that the dataset is BIDS formatted for processing with fMRIPrep

.. code-block:: console

   bidskit

Singularity will by default make environment variables available to the container, which can expose unintended software to fMRIPrep (e.g., the user's FSL version instead of the container's built-in version).
Use the ``--cleanenv`` flag to avoid passing environment variables.

However, you do need to pass an environment variable containing the path to the freesurfer license file. Do this by creating a ``APPTAINERENV_FS_LICENSE`` variable, which singularity will pass to the container as ``FS_LICENSE``.
After specifying the path to the license file, e.g. ``/opt/fslicense/license.txt``, you must also bind the folder on the host system to this new path. The option ``--bind /usr/local/fslicense:/opt/fslicense`` will map the contents of ``/usr/local/fslicense`` on the host to the path ``/opt/fslicense`` within the container.

fMRIPrep makes extensive use of a temporary directory for intermediate files. Depending on system configuration, the contents of ``/tmp`` may not be sufficient, causing the processing job to halt and error before completing. To remedy this, map a different location on the host system unrestricted in size to ``/tmp``, e.g. ``$HOME/tmp:/tmp``.

*NB: Clear this custom tmp directory regularly when you no longer need the intermediate working files.

If fMRIPrep aborts partway through, you can rerun it from where it stopped by using the ``-w`` option to instruct fMRIPrep which existing work directory to use. Note, this refers to the work directory within the container, i.e. ``/tmp/work``, not the path to the work directory on the host sytem.


.. code-block:: console

   conda activate fmriprep-env

   export APPTAINERENV_FS_LICENSE=/opt/fslicense/license.txt
   singularity run --cleanenv --bind /usr/local/fslicense:/opt/fslicense,$HOME/tmp:/tmp /usr/local/singularity/nipreps_fmriprep_23.2.0-2024-01-10-63081a7fe2b8.simg \
       $HOME/path/to/bidsdata $HOME/path/to/bids/derivatives \
       participant \
       --me-output-echos \
       -w /tmp/work
