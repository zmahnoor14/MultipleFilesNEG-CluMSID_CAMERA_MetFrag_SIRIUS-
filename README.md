# Multiple Files (LC-MS/MS Metabolome Annotation Workflow)
Metabolome Annotation Workflow takes multiple .mzML spectral files and provides candidates per m/z and per compound predicted by [MetFrag](https://ipb-halle.github.io/MetFrag/) and [SIRIUS](https://bio.informatik.uni-jena.de/software/sirius/) respectively.

## Summary of the Workflow
The objective of this Workflow is to perform Metabolite Annotation, specifically dereplication of known-known compounds in the spectral files. The Dereplication is performed using the following Databases (using the Metfrag and SIRIUS softwares):<br>PubChem, KEGG, HMDB, MeSH, KNApSAck, ChEBI, HSBD, MaConDa, MetaCyc, GNPS, ZincBio, UNDP, YMDB, PlantCyc, NORMAN.<br> <br> This workflow is divied into three parts.<br> 
* Reading MS2 spectra (.mzML files) and merging the Spectra that belong to one precursor m/z **(CluMSID)**.
* Adduct Annotation, finding out same compound m/z values and extracting neutral precursor m/z values **(CAMERA)**.
* Metabolite Identification (Dereplication) for each precursor m/z **(MetFrag & SIRIUS)**.

## Input
The second block of the code, after loading the libraries, is the input cell. Following input variables are important to set the correct directories:
```
path_name ##here add the full directory to the folder where you want to keep the results
input_path_name ##here add the full directory to the folder where you have kept the input files
```
Before we move onto the inputs, it is important to note that naming of the input files is very important, which will be explained in this section. <br> <br> There are three types of input files for this workflow: 
* MS2 Spectral files; note that these .mzML files must contain precursor masses and their fragment spectral information.
```
#IMPORTANT: name all the mzml files using e.g: a project name and then number the files starting from 01 to onwards. For pattern, add the project name so that all the mzml files from that project are stored as a list in this variable

mzml_files <- list.files(input_path_name, pattern = "insert_the_pattern_for_files")

##this is important so that the mzml_files list contains the names along with the directory, to avoid any errors regarding the directory of inputs
mzml_files <- paste(input_path_name, mzml_files, sep ="")
```
* Quality Control file; in this workflow, I have used one general QC file, which contains accurate precursor m/z and aids the annotation/ dereplication process, as it is generated with high resolution. Note that this is also a spectral file with .mzML extention
```
qc_file <- paste(input_path_name, "insert_QCfile_name.mzML", sep ="")
```
* Inclusion lists (.csv); these files are important for the workflow. For each MS2 spectral file (Project_01.mzML), there is corresponding inclusion list (Project_InclusionList_01.csv) which contains information about precursor mass, the start and end retention time in minutes and the polarity. For example:

| Mass[m/z] | Polarity | Start[min] | End[min] |
| ----------- | ----------- | ----------- | ----------- |
| 84.06544 | Positive | 5.84 | 6.84 |
| 102.26589 | Negative | 10.57 | 11.57 |

These columns are important for the workflow.
```
#IMPORTANT: name all the inc files using e.g: a project name and then number the files starting from 01 to onwards (just like the numbering for MS2 spectral files; 01.mzML should be the MS2 spectra from inclusionlist01.csv ). For pattern, add the project name so that all the inc files from that project are stored as a list in this variable

inc_files <- list.files(input_path_name, pattern = "insert_the_pattern_for_inclusionlist_files")

##this is important so that the inc_files list contains the names along with the directory, to avoid any errors regarding the directory of inputs

inc_files <- paste(input_path_name, mzml_files, sep ="")
```
The resulting input_files dataframe will look like this.

| mzml_files | inc_files |
| ----------- | ----------- |
|/Usr/ProjectInputFiles/Project_01.mzML | /Usr/ProjectInputFiles/InclusionList_01.csv |
| /Usr/ProjectInputFiles/Project_02.mzML | /Usr/ProjectInputFiles/InclusionList_02.csv |

## Installations
### R-packages for metabolomics (BiocManager Version 1.30.10)
#### CluMSID Version 1.2.1
GitHub: <https://github.com/tdepke/CluMSID>. The Workflow starts with CluMSID which performs two functions: it extracts the MS2 Spectra and merges the MS2 spectra (here, according to the features present in inclusion list).
```
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install("CluMSID")
library(CluMSID)
```
#### CAMERA Version 1.42.0
GitHub: <https://github.com/sneumann/CAMERA>. Next is CAMERA which performs the isotope and adduct annotation. It also groups the features possibly coming from same compound and provides information on the neutral precursor mass (neutral precursor mass information is used by MetFrag later)
```
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install("CAMERA")
library(CAMERA)
```
#### Others
* dplyr Version 1.0.2
```
install.packages(dplyr)
library(dplyr)##### use to filter m/z from same compound
```
* stringr Version 1.4.0
```
install.packages(stringr)
library(stringr)#### for removing string with certain characters
```
* matrixStats Version 0.57.0
```
install.packages(matrixStats)
library(matrixStats)##### to calculate median for rt in sec from inclusion list
```

### Metabolome Annotation Softwares (CommandLine)
#### MetFrag
GitHub: <https://github.com/ipb-halle/MetFrag>. MetFrag is a software for annotation of high precision LC-MS/MS data. In order to use MetFragCL, download the jar file from <https://ipb-halle.github.io/MetFrag/projects/metfragcl/> <br>
**Important** Keep this jar file (MetFrag2.4.5-CL.jar) in the Result folder (under the name of: path_name variable described in the input section)
#### SIRIUS
GitHub: <https://github.com/boecker-lab/sirius>. Sirius is a toolkit to peform metabolomics analysis on LC-MS/MS data. Tools used in this workflow are Formula Identification tool, CSI:FingerID and CANOPUS. The version I used is SIRIUS 4.5.1 which has been updated to 4.6.1 (this shouldn't cause any problem, but report if it does). It is important to download the latest version and not the old version 4.0.1<br> 
To sucessfully install SIRIUS, follow as mentioned (Mac users):
* download the SIRIUS+CSI:FingerID GUI and CLI zip folder from <https://bio.informatik.uni-jena.de/software/sirius/>
* Double click the zip file to unzip. Now right-click the 'sirius-gui'
 application file and click on 'Show Contents'. In this folder, you will find another folder named as MacOS, which contains the command line 'sirius'. 
 * To use this command line, add the following path to either .bash_profile or .zprofile. You can find these files easily using [FileZilla](https://filezilla-project.org/) (FTP client, available for Mac, Windows and Linux) and then opening your user name folder. There are other ways too using terminal. When you open your .bash_profile, add this to the $PATH variable:
 ```
 PATH="/Users/name/Projectfiles/sirius.app/Contents/MacOS/:${PATH}"
export PATH
```
Here instead of /Users/name/Projectfiles, add you directory for 'sirius'. I have described above how to reach to this 'sirius' file. Now, open terminal and just type ```sirius```. If it runs, then you are all set to use the code and SIRIUS. If not, then check other bash scripts to add the path variable to and check on terminal again.

## Usage of the Workflow
You need either R, R-Studio or R kernel on Jupyter Notebook for this workflow. In case your Jupyter Notebook doesn't have R kernel, here's how to install it:  [Install R kernel on Jupyter Notebook](https://simply-python.com/2019/06/24/running-r-on-jupyter-notebook-with-r-kernel-no-anaconda/).
1. Open the Jupyter Notebook and run R kernel
2. In this GitHub repository you will find two jupyter notebooks named: **MultipleFilesNEG(CluMSID_CAMERA_MetFrag_SIRIUS).ipynb** for negative mode files and **MultipleFilesPOS(CluMSID_CAMERA_MetFrag_SIRIUS).ipynb** for positive mode files. Follow the notebook according to the mode of your input files.
3. Install CluMSID, CAMERA, dplyr, stringr and matrixStats as described above and load these packages.
4. Now, set inputs which are in the second cell of the notebook. First is the path_name which should be the directory where you want to store the results; this same directory **should** contain the MetFrag jar file named as MetFrag2.4.5-CL.jar. Next is the input_path_name which is the directory containing all your inputs; MS2 spectral files (.mzML), QC file (.mzML) and all inclusion lists (.csv). Follow the input instructions given above.
5. The third and final cell can be then used without any modifications. Details of each function and loop is described in comments throughout the code.

## Notes
For now, this workflow is only available as a code. Note that this workflow is not for LC-MS spectral files; it is specific for LC-MS2 spectral files, because our aim is to perform Metabolome Annotation. Hence, there are **NO** preliminary steps for retention time correction, grouping and alignment of the m/z values over various samples.

## References
<a id="1">[1]</a> 
Depke, Tobias & Franke, Raimo & Brönstrup, Mark. (2019).
CluMSID: an R package for similarity-based clustering of tandem mass spectra to aid feature annotation in metabolomics. 
Bioinformatics. 

<a id="2">[2]</a>
Kuhl, C., Tautenhahn, R., Böttcher, C., Larson, T. R., & Neumann, S. (2012). CAMERA: an integrated strategy for compound spectra extraction and annotation of liquid chromatography/mass spectrometry data sets. Analytical chemistry, 84(1), 283–289.

<a id="3">[3]</a>
Ruttkies, C., Schymanski, E.L., Wolf, S. et al. (2016). MetFrag relaunched: incorporating strategies beyond in silico fragmentation. J Cheminform 8, 3.

<a id="4">[4]</a> 
Kai Dührkop, Markus Fleischauer, Marcus Ludwig, Alexander A. Aksenov, Alexey V. Melnik, Marvin Meusel, Pieter C. Dorrestein, Juho Rousu and Sebastian Böcker. (2019).
SIRIUS4: a rapid tool for turning tandem mass spectra into metabolite structure information.
Nat Methods.
