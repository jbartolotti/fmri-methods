3D Printing Brains from Anatomical MRI
=====================

.. _printbrain:

Anatomical to Printable
--------------------

The Synapse command ``make3dBrain`` converts anatomical files to 3d-printable .stl files using freesurfer. It accepts a single participant at a time, and you can provide any one of the following: 
 * A directory containing DICOM files from a T2 anatomical scan
 * A single NIFTI file containing the anatomical image
 * A freesurfer output directory containing ``lh.pial`` and ``rh.pial`` files.

Specify an output directory and filename, and the script will create a folder called ``FILENAME`` containing the two files ``FILENAME_lh.stl`` and ``FILENAME_rh.stl``. 

Note: If you supply DICOM or NIFTI files, processing will be a multi-hour job due to freesurfer reconstruction time. You may optionally supply a KUMC email address and you will be notified by email when complete, along with instructions for further processing and printing. If you supply a non-KUMC address, your email will be sent to an admin who will attempt to forward it to you.

Usage: 

.. code-block:: console

   make3dBrain /myproject/SCANS/1/DICOM /path/to/3dPrints MyBrain myemail@kumc.edu

for additional help, run ``make3dBrain -h``

Refining the Printable .SLT
--------------------------

The command ``make3dBrain`` outputs two printable .stl files, for the left and right hemispheres. Before printing, we need to simplify the model and combine the two hemispheres into a single print. See https://www.instructables.com/3D-print-your-own-brain/ for additional details.
These instructions are written for MeshLab, which you can download at https://www.meshlab.net/#download
•	Start MeshLab and import MyBrain_lh.stl and My_Brain_rh.stl via ‘File’ -> ‘import mesh’.
•	Click on 'Filters' -> 'Mesh Layer' -> 'Flatten Visible Layers' and apply. Now your brain mesh is RH_LH combined. 
•	Simplify vertices before printing with 'Filters' -> 'Remeshing, Simplification, Reconstruction' -> 'Simplification: Quadratic Edge Collapse Decimation'". Enter the number of faces to use for simplification; try 100,000. Click apply.
•	Next, apply smoothing. Use ‘Filters’ -> ‘Smoothing, Fairing and Deformation’ -> ‘Laplacian Smooth’ with 3 steps. Click the apply button once. Adjust these settings for different amounts of smoothing. HC Laplacian Smooth also gives good results.
•	When done, use ‘File’ -> ‘Export Mesh As’ and save as an .stl file for printing.
