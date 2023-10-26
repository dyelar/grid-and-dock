# Grid and Dock


This program is meant to help with running the necessary command line programs for mgltools, autogrid, and autodock from the command line.  The program makes assumptions about the locations of where mgltools (/opt/mgltools/), autodock (/usr/bin/autodock), and autogrid (/usr/bin/autogrid) are installed within the environment.  It should properly handle all of the necessary work to execute the python 2 scripts from mgltools installation.  


## Single Job
```

Usage:  grid-and-dock ligand.pdbqt receptor.pdbqt grid-template.in results-directory [job-identifier] 

	results-directory: 	 The location of all files associated with this job. 
	job-identifer: 		 [Optional] A text tag to identify the job [default = ligand filename].
 

An output log file called output.log will be in the results directory, along with all files associated with job.

```

The program will put all of the input files into the result's directory, so that everything is kept in one place.  This tends to help when trying to come back later and review the results, or when the program is run in parallel.  The output.log file should contain information about what arguments the program was started with and all of the outputs from the commands executed from this program to maximize the ability find problems upon a failure.
.

### Error Handling

This program attempts to catch all errors and properly fail at the step where the error occurred, while capturing all outputs from the programs.

## Parallel Runs

Multiple scripts have been created to help faciliate running the above for simultaneous runs with different ligands and receptors/grid files.  Some helper scripts have also been created to help with generating the input files to these scripts.

### Path Requirements

You will need to add the directory containing these programs to your PATH, since some of the programs reference other programs.  The command names have been made long and descriptive to help prevent collisons within your PATH


### Input File Generation Scripts

#### Ligand List
The program _make-ligand-list_ is passed a directory containing only .pdbqt files and the name/location of the output file you want to generate.  It ignores directories within passed directory.  It puts absolute paths into the ligand list file.

#### Receptor List
The program _make-receptor-list_ is passed a directory containing receptor files with a .pdbqt extension and grid files with a .gpf extension.  This program assumes that the portion of the file name before the extension match between the receptor and grid parameter file.  If this is not the case, then you will need to generate the file using another method.  The file uses absolute paths.  An example of the file format is below here:

```
receptor-file","reference-grid-parameter-file"
"/data/receptors/90000125.pdbqt","/data/receptors/90000125-but-I-am-just-a-bit-different.gpf"
"/data/receptors/90000187.pdbqt","/data/receptors/90000187.gpf"
"/data/receptors/90000301.pdbqt","/data/receptors/90000301.gpf"
```

### Parallel running programs

Each of these scripts is used for running the jobs in parallel with a number of active threads indicated on the command line after the ligand, receptor, and grid files.  The final argument is a base results directory for where results should be placed.  If no directory is provided, then the base directory is the current working directory.  Efforts have been made attempt to allow for all kinds of special characters within filenames, except for a null '\0' character, which is used internally for separate the different arguments.

Programs:

* **run-multiple-ligand-single-receptor** - _Arguments:_ ligand-list.txt receptor.pdbqt grid-file.gpf number-of-threads base-results-directory
* **run-multiple-ligand-multiple-receptor** - _Arguments:_ ligand-list.txt receptor-and-grid-list.txt number-of-threads base-results-directory
* **run-single-ligand-multiple-receptor** - _Arguments:_ ligand.pdbqt receptor-and-grid-list.txt number-of-threads base-results-directory (**NOTE:**  Unlikely to see much much use, but here for completeness sake)

### Output Structure

Since you could be potentially running with many threads and each run generates multiple files, it was important to create a directory structure to prevent collision and to make it so that a simple _ls_ afterwards wouldn't potentially take minutes to respond.  In the case of multiple ligands and a single receptor, then the first four characters of the ligand filename will be used to create a directory and within that directory will be a directory containing the entire ligand filename without the extension.  This directory will have all of the inputs and outputs within it.  If a single ligand and multiple receptors is run, then you have the same structure with receptor replacing the ligand.  If you are running multiple ligands and receptors, then the you will see the multiple ligand structure within the multiple receptor structure.  (For Example:  r123/r12345/l987/l987654/)

#### Output Completion Status

The programs will output to the standard output a message saying whether that job failed or completed, based on the filename of which has multiples.  In the case where both have multiples, it uses the receptor-ligand as output.  In all cases, the extension is removed.

