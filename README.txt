SMaPPOI installation and usage notes
v1.0 (26 Apr. 2017)
J. Peter Rickgauer
rickgauerj@janelia.hhmi.org


INSTALLATION

1. Create a new directory (the "install directory") and extract the contents of the archive into that directory.
2. Append the install directory to the system path.
3. Install the Matlab Runtime Libraries using the following procedure (installation is required to run compiled versions of the code):
	a. Download the R2014b MCR installer zipfile from the Mathworks website (http://www.mathworks.com/products/compiler/mcr) into the "MCR/" directory within the install directory.
	b. Go to the MCR subdirectory and unzip the installer zipfile.

	c. Install the libraries by running the following from the command line, after replacing <absolutePathToPwd/> with the full absolute path to the MCR subdirectory as appropriate:

./install -mode silent -agreeToLicense yes -destinationFolder </absolutePathToPwd/> -outputFile <absolutePathToPwd/>

	d. once installation finishes, the installer should display several lines of instructions that tell you how to change your environment variables ($LD_LIBRARY_PATH and $XAPPLRESDIR). Follow those instructions, then restart the command shell (or source your .rc file).

	e. If this does not work, go to https://www.mathworks.com/help/compiler/install-the-matlab-runtime.html for other ideas.

4. Test the code using sample files included in the archive as described below in "EXAMPLES".



SYNTAX

./smappoi_run.sh <function> <paramFile>

<function> can be either calcSP or searchImage
<paramFile> is an ASCII file with parameters appropriate to the function being run (see below).

smappoi_run.sh is a shell script that starts instances of the programs using the requested number of cores, passing along parameters given in the parameter file.



EXAMPLES

In the following, "$ID" is used to represent the install directory. 

1. Calculate the electrostatic and scattering potential matrices for a known reference structure (apoferritin, PDB:2W0O).
	a. Open $ID/samples/sample_calcSP.par and modify the line that reads "nCores 32" to match the number of available CPU cores that you wish to use in the calculation (e.g., "nCores 16"). Save and close the file.
	b. From $ID, start the calculation by running:

	./smappoi_run.sh samples/sample_calcSP.par

	c. When the calculation is complete, verify that there are 3 files in $ID/samples/output/: 1) the electrostatic potential; 2) a padded scattering potential; and 3) a concatenated log file.

Using 32 CPUs (Intel(R) Xeon(R) CPU E5-2698 v3 @ 2.30GHz), this calculation should take about 5 minutes. 


2. Search a simulated image to detect the location and orientation of apoferritin. 
	a. Open $ID/samples/sample_searchImage.par and modify nCores if appropriate. Verify that the file paths are correct.
	b. From $ID, start the calculation by running:

	./smappoi_run.sh samples/sample_searchImage.par

	c. When the calculation is complete, verify that there are 9 additional files in $ID/samples/output/ beginning with the word "search" (described below). 

Using 32 CPUs (Intel(R) Xeon(R) CPU E5-2698 v3 @ 2.30GHz), this calculation should take about 1-2 minutes. This search only includes a subset of the rotations necessary to cover all of SO3; to search the full rotation space, change the "rotationsFile" parameter to "rotations/hopf_R4.txt" or "rotations/hopf_R5.txt" in the .par file.

Output files generated from a search:

		i. searchImage.mrc: the image used for cross-correlation (whitened if psdFilterFlag is set to 1)
		ii. search_mip.mrc: maximum intensity projection (MIP) of all cross-correlation (CC) values at each pixel in the image 
		iii. search_mipi.mrc: indices of rotation matrices at each pixel in the MIP that produced the corresponding CC value
		iv. search_sumImage.mrc: sum of all CC values (first moment)
		v. search_squaredSumImage.mrc: sum of all squared CC values (second moment)
		vi. search_listByRotation.dat: rotation-indexed list of CC maxima and corresponding pixel in the image
		vii. search_listAboveThreshold.dat: list of all CC values above the threshold specified by "arbThr" in the parameter file
		viii. search_hist.dat: 1024-bin histogram of all CC values within the range specified by "binRange" in the parameter file
		ix. search.log: log file



PARAMETER FILES

calcSP expects the following input parameters: projectDir, PDBFile, aPerPix, V_acc, jobsPerRun. 

function calcSP
outputDir samples/output
PDBFile samples/2W0O.pdb
aPerPix 0.965
V_acc 300000
nCores 32
# any line with a comment is ignored

outputDir: directory for output files
PDBFile: name of input .pdb file
aPerPix: voxel pitch (in Angstroms)
V_acc: microscope accelerating voltage (in V)
nCores: # of CPU cores to use for calculation

searchImage expects the following input parameters:

function searchImage
outputDir samples/output
imageFile samples/testImage.mrc
modelFile samples/output/2W0O_SP.mrc
rotationsFile rotations/hopf_R4.txt
aPerPix 0.965
defocus 70 70 0
MTF 0 0.9350 0 0 0.6400
F_abs 0.0
arbThr 5
psdFilterFlag 0
Cs 0.0027
Cc 0.0027
deltaE 0.7
V_acc 300000
a_i 0.000050
nCores 32
binRange 15

outputDir: directory for output files
imageFile: name of input image to search (.mrc)
modelFile: name of scattering potential volume to use for template generation (.mrc)
rotationsFile: name of file including rotation matrices specifying template orientations
nCores: # of CPU cores to use for calculation
binRange: binning range for CC histograms (in SNR units, or standard deviations above the mean for what is expected of a Gaussian noise distribution of similar size). 
-- microscope parameters to assume in template generation: --
aPerPix: voxel pitch (in Angstroms)
defocus: astigmatic defocus (see Rohou and Grigorieff, JSB 2015)
MTF: detector modulation transfer function parameters a, b, c, alpha, beta (see Rullgard et al., J. Microscopy 2011)
F_abs: amplitude contrast coefficient (0-1.0)
arbThr: threshold CC value above which all values are saved (with corresponding pixel coordinates and rotation matrix indices)
psdFilterFlag: apply a whitening filter based on the reciprocal square root of the radially averaged power spectral density of the image
Cs: spherical aberration coefficient (in m)
Cc: chromatic aberration coefficient (in m)
deltaE: energy spread of the source (in eV)
V_acc: microscope accelerating voltage (in V)
a_i: illumination aperture (in radians)


FILES INCLUDED

# source code:
src/calcSP.m 
src/searchImage.m
ba_interp3.m (see REFERENCES)
ba_interp3.cpp (see REFERENCES)
ba_interp3.mexa64 (see REFERENCES)
ba_interp3.mexmaci64 (see REFERENCES)

# shell script to run calcSP and searchImage binaries:
smappoi_run.sh

# binaries (compiled with Matlab Compiler mcc v5.2, R2014b):
calcSP # calculates elastic scattering potential matrix for a molecule, given its .pdb file
searchImage # given a scattering potential, calculates projection templates for a given set of microscope properties and cross-correlates them against an image

# sample files:
samples/sample_calcSP.par # sample parameters file for calcSP function
samples/sample_searchImage.par # sample parameters file for searchImage function
samples/2W0O.pdb # sample PDB file (apoferritin)
samples/testImage.mrc # simulated image of apoferritin at 70 nm underfocus, 10.7 e/A^2, rotated to orientation #100 in the reduced rotations set (rotations_hopf_R4_1k.mat)
samples/testMIP.mrc # one expected output file from searching sampleImage.mrc for 2W0O

# rotation matrix sets:
rotations/hopf_R5.txt # 2,359,296 rotation matrices generated using the hopf fibration (see REFERENCES)
rotations/hopf_R4.txt # 294,912 rotation matrices generated using the hopf fibration (see REFERENCES)
rotations/hopf_R4_1k.txt # subset of those rotation matrices (#1-1,000)



REFERENCES

This code makes use of modified versions of the following external code:
1. ba_interp3 - B. Amberg; File ID #21702, https://www.mathworks.com/matlabcentral/fileexchange/
2. ReadMRC and associated code - F. Sigworth; File ID #27021, https://www.mathworks.com/matlabcentral/fileexchange/
3. fastPDBread - https://www.mathworks.com/matlabcentral/fileexchange/35009-a-fast-method-of-reading-data-from-pdb-files/all_files
4. bindata (P. Mineault; https://xcorr.net/?p=3326, http://www-pord.ucsd.edu/~matlab/bin.htm
5. parametrizeScFac.m from InSilicoTEM, as described in Vulovic M. et al., Image formation modeling in cryo-electron microscopy, J. Struct. Biol. (2013) 183(1):19-32.
6. Rotation matrix sets were generated using code referenced in: Generating Uniform Incremental Grids on SO(3) Using the Hopf Fibrations, International Journal of Robotics Research, IJRR 2009 Anna Yershova, Swati Jain, Steven M. LaValle, and Julie C. Mitchell.


