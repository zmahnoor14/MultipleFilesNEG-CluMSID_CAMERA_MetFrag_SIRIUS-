# MultipleFilesNEG-CluMSID_CAMERA_MetFrag_SIRIUS-
Metabolite Annotation workflow (Multiple files in one ionization mode (e.g: here negative) using CluMSID_CAMERA_MetFrag_SIRIUS)

## Introduction
This Workflow is developed with the objective to perform Metabolite Annotation from the Mass Spectrometry Files (mzml). <br> Here, Dereplication is executed with MetFrag and SIRIUS. So, it is designed for obtaining results for known-known Metabolites.

## Input
There should be "Three" input files: input mzml files and the QC file. Give a variable to both kind of files. Third is the inclusion list, This workflow specifically takes inclusion list to merge MS2 Spectra, hence you need inclusion list as csv. (if code modified,you can merge MS2 spectra without inlcusion list)

## R-packages used
### CluMSID
GitHub: https://github.com/tdepke/CluMSID. The Workflow starts with CluMSID which performs two functions: it extracts the MS2 Spectra and Merges the MS2 spectra (here, according to the features present in inclusion list).
### CAMERA
GitHub: https://github.com/sneumann/CAMERA. Second is CAMERA which performs the isotope and adduct annotation. It also groups the features possibly coming from same compound.
### MetFrag
GitHub: https://github.com/ipb-halle/MetFrag. MetFrag uses the information from the above two packages and performs dereplication against KEGG and PubChem.
### SIRIUS
GitHub: https://github.com/boecker-lab/sirius. SIRIUS performs similar functions but against many databases

