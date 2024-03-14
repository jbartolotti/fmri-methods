Preprocessing fMRI data with fMRIPrep
================

.. _setup:

Setup
--------------

**IMPORTANT:** These installation steps should already be done for you if using an HBIC managed environment, e.g. Synapse.

Install `Singularity <https://docs.sylabs.io/guides/3.0/user-guide/installation.html>`_ and `Conda <https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html>`_

Build fmriprep singularity image in ``/usr/local/singularity`` with the appropriate version number (see version list `here <https://fmriprep.org/en/stable/changes.html>`_)

.. code-block:: console

  singularity build /usr/local/singularity/fmriprep_<version>.simg \
                    docker://nipreps/fmriprep:<version>

Create a new conda environment for `bidskit <https://github.com/jmtyszka/bidskit/tree/master>`_

.. code-block:: console

  conda create -n bidskit-env numpy
  conda activate bidskit-env
  pip install pydicom bidskit dcm2niix

Obtain a `freesurfer license <https://surfer.nmr.mgh.harvard.edu/fswiki/License>`_ and place in a folder to bind in the singularity container, e.g. ``/usr/local/fslicense/license.txt``


| *(Note: tedana is required for multiecho processing only)*
| Create a new conda environment for tedana. Use python 3.8 due to compatibility issues between mapca and the sklearn version (1.4) included in higher python versions.

.. code-block:: console

  conda create -n tedana-env python=3.8 nilearn nibabel numpy scikit-learn scipy mapca
  conda activate tedana-env
  pip install tedana



.. _bids:

BIDS Format
------------

`BIDSKIT <https://github.com/jmtyszka/bidskit/tree/master>`_ is one of several utilities available for converting DICOM images to BIDS neuroimaging formats.

First, create a dataset folder, e.g. ``/path/myproject`` that contains a single ``sourcedata`` folder. Within ``sourcedata``, create a folder for each participant. In the participant folder, create a folder for each session that contains the DICOM images. 

.. code-block:: console

  myproject/sourcedata/1001/1/[DICOM Images]
                           /2/[DICOM Images]
                      /1002/1/[DICOM Images]
                           /2/[DICOM Images]

The first-pass conversion by *bidskit* will create a translator text file that you will edit to aid in creating the BIDS-compliant structure. This step only needs to be done once per project. After activating the ``bidskit-env`` conda environment, you can run *bidskit* from the data directory (i.e., the folder that contains the ``sourcedata`` subfolder) or by specifying its location. If you have only one scanning session, use the --no-sessions option, ``bidskit --no-sessions`` in both the first and second bidskit calls.

.. code-block:: console

  conda activate bidskit-env
  cd /path/myproject
  bidskit

.. code-block:: console

  conda activate bidskit-env
  bidskit -d /path/myproject

Navigate to ``/path/myproject/code`` and open ``Protocol_Translator.json`` for editing in a text editor. Make use of BIDS documentation to aid in editing this file to assign appropriate BIDS purpose directory names (anat, func, fmap, etc.) and BIDS-compliant filename suffixes. 

- https://reproducibility.stanford.edu/bids-tutorial-series-part-1a/
- https://bids-specification.readthedocs.io/en/stable/introduction.html
- https://bids-standard.github.io/bids-starter-kit/tutorials/annotation.html
- https://bids-standard.github.io/bids-examples/#mri
- https://andysbrainbook.readthedocs.io/en/latest/OpenScience/OS/BIDS_Overview.html
- https://bids.neuroimaging.io/

After completing ``Protocol_Translator.json``, run *bidskit* a second time using the same syntax as before and it will create the appropriate ``/path/project/sub-XXXX`` folders with NIFTI files and .json descriptors with BIDS-compliant filename structures.

**IMPORTANT:** If using multiecho data, on the second call to *bidskit*, add the ``--multiecho`` option so that ``dcm2niix`` creates individual files for each echo. **In addition**, add this line to /path/myproject/.bidsignore: ``*echo*T1W*``. Echo number for T1W images was recently allowed in the BIDS standard, but it `may still throw an error <https://github.com/bids-standard/bids-specification/issues/654>`_ in some BIDS verifiers.

.. code-block:: console

  bidskit --multiecho

After the second pass of *bidskit*, you will still need to supply additional information not contained within the DICOM images. You will need to edit ``dataset_description.json``, ``participants.json``, ``participants.tsv``. In addition, for task-based functional sessions you will need to edit any ``/path/myproject/sub-XXXX/ses-X/func/*_events.tsv`` template files with task timing information.

.. _fmriprep:

fMRIPrep 
-------------

`fMRIPrep <https://fmriprep.org/en/stable/>`_ is a robust, easy to use application for preprocessing task-based and resting-state fMRI. For more details on the steps fMRIPrep performs, see the `workflow description <https://fmriprep.org/en/latest/workflows.html>`_. It is highly recommended that fMRIPrep be run within a container environment to standardize the software packages used. This guide uses fMRIPrep in a Singularity container.

Singularity will by default make environment variables available to the container, which can expose unintended software to fMRIPrep (e.g., the user's FSL version instead of the container's built-in version).
Use the ``--cleanenv`` flag to avoid passing environment variables.

However, you do need to pass an environment variable containing the path to the freesurfer license file. Do this by creating a ``APPTAINERENV_FS_LICENSE`` variable, which singularity will pass to the container as ``FS_LICENSE``.
After specifying the path to the license file, e.g. ``/opt/fslicense/license.txt``, you must also bind the folder on the host system to this new path. The option ``--bind /usr/local/fslicense:/opt/fslicense`` will map the contents of ``/usr/local/fslicense`` on the host (where the file actually resides) to the path ``/opt/fslicense`` within the container (where fMRIPrep will look for it, that is, the path specified by ``FS_LICENSE``).

fMRIPrep makes extensive use of a temporary directory, ``/tmp`` for intermediate files. ``/tmp`` is automatically bound from the host system to the container, so fMRIPrep will use ``/tmp`` on the host for its working directory. Depending on system configuration, ``/tmp`` may not have sufficient disk space allotted, causing the processing job to halt and error before completing. To remedy this, map a different location on the host system that is unrestricted in size to ``/tmp``, e.g. ``$HOME/tmp:/tmp``. fMRIPrep will then put its working directory in ``$HOME/tmp/work`` on the host system.

*NB: Clear this custom tmp directory regularly when you no longer need the intermediate working files.*

If fMRIPrep aborts partway through, you can rerun it from where it stopped by using the ``-w`` option and passing the work directory from the failed attempt. Note, this refers to the work directory within the container, i.e. ``/tmp/work``, not the path to the work directory on the host sytem.

**Multi-echo preprocessing:** You must add the option ``--me-output-echos`` for fMRIPrep to write processed files for each echo that you can optimally combine with tedana.

The syntax for running fMRIPrep in a singularity container is generally:

.. code-block:: console

   singularity run [singularity options] /path/to/container/fmriprep<version>.simg bids_dir output_dir participant [fMRIPrep options]

- ``bids_dir`` is the root folder of a BIDS valid dataset (sub-XXXX folders should be found at the top level in this folder).

- ``output_dir`` is where the preprocessed files and reports will be saved

.. code-block:: console

   export APPTAINERENV_FS_LICENSE=/opt/fslicense/license.txt
   singularity run --cleanenv --bind /usr/local/fslicense:/opt/fslicense,$HOME/tmp:/tmp \
       /usr/local/singularity/fmriprep_23.2.0.simg \
       $HOME/path/to/bidsdata $HOME/path/to/bidsdata/derivatives \
       participant \
       --me-output-echos \
       -w /tmp/work

.. _tedana:

tedana
-------------

`Tedana <https://tedana.readthedocs.io/en/stable/index.html>`_ is a python package for preprocessing multiecho data to obtain an optimally combined image. This image can then be used in further analyses the same way as a single-echo preprocessed image. The advantage of the ME image is that non-BOLD signals are minimized in the optimally combined image compared to single-echo.

**Note:** tedana currently *requires* sklearn version 1.2. the library mapca expects PCA objects to have the attribute `n_features_` which was removed as of sklearn 1.4 and replaced with `n_features_in`. Use a conda environment with python 3.8 to use sklearn 1.2.

From the command line, tedana requires the multi-echo files and the echo times in ms.

.. code-block:: console

    tedana -d /path/to/echos/echo*.nii.gz -e 15.0 39.0 63.0 [options]



.. code-block:: console

    conda activate tedana-env
    tedana -d /path/to/bids/derivatives/sub-XXXX/ses-XX/func/sub-XXXX_ses-XX_task-XXXX_echo-*_desc-preproc_bold.nii.gz     \
          -e 13.0 30.99 48.98 66.97 84.96    \
          --out-dir /path/to/bids/derivatives/sub-XXXX/tedana

For further analysis, you will generally be using the optimally combined image, found in the output directory you specified, e.g. ``/path/to/bids/derivatives/sub-XXXX/tedana/desc-optcom_bold.nii.gz``


