*Current version Sept 1, 2023*

## Purpose

This document provides tips on using [UAHPC](https://oit.ua.edu/service/user-portal/) for neuroimaging analysis. The contents of this file would be revised & extended.

## Disclaimer & Credits

This document was developed by Dr. Firat Soylu (fsoylu@ua.edu) based on notes taken during use of UAHPC for neuroimaging analysis. The local computer used during these analyses was a Ubuntu 18.04 LTS laptop, therefore you might need to take additional steps in running the steps described with another local OS (MAC & Windows; most likely to be same for all other Linux distros). This is in no way a comprehensive guide (might become one day if other people contribute to it) Please use the document at your own risk. The author accepts no responsibility if use of this document leads to loss of data or any other unintended results (including bad science; that would be definitely your fault). Please read UAHPC documentation or contact UAHPC for the most authorative responses to your questions.

## Helpful Resources

SLURM documents: https://slurm.schedmd.com/quickstart.html
UA HPC: https://oit.ua.edu/service/research-getting-started/

These are also helpful resources, though the information provided above may not be entirely compatible with UAHPC:

https://www.chpc.utah.edu/documentation/software/matlab.php

https://hpcc.usc.edu/support/documentation/matlab/

## UAHPC Storage Folders

### `home/username` : 20 GB, personal space

This is the users home directory.

### `/grps2/alri` : 10 TB, long term storage

This is the shared ALRI storage place. We have only 10 TB. Even though this might look big, it is not. This should be used for permanent storage. It is backed up \[check?\]

### `/bighome` :unlimited, short-term/temprorary

This is a temprorary workspace, ideal location for conducting analysis. The contents of folders in this spaced are deleted after a few weeks \[check?\]. Eventually files that need to be permanently stored can be moved to the `/grps2/alri` folder.

## Software on UAHPC

MATLAB, Freesurfer, SPM, fsl, mricron, Slicer 3D ... are already installed. New software can be requested. Alternatively software in the form of MATLAB toolboxes can be installed in the user's home directory & added to the MATLAB path. The same applies to SPM and its toolboxes (e.g., CAT12).

## How to Connect to UAHPC

### GUI use

If you need to open the GUI of a software running on UAHPC on your local computer (e.g. laptop) then connect to the *main node* using:

`ssh -X username@uahpc.ua.edu`

- `-X` directs the x-server output to the host computer
- For use in Windows *xming* (x-server for Windows) and *Putty* (SSH client) need to be installed. Linux works as is. Again, because the analysis covered here was run on Linux no additional help is provided here for running it on Windows.

10/19/2020: I was told that (by HPC people) I should use an interactive compute node to run matlab. This (8 cores & 40 GB memory) was suggested:

`srun -c 8 --mem-per-cpu=5G -p main -q main --pty bash`

## Basic Commands

**Load software**
Before you use a software on UAHPC, it needs to be loaded:
`module load matlab` (use of `use` is not recommended)

**Request node**:
Once you connect you are in the *main node*. This is to be used for basic testing & scripting and not for running your full analysis. I was told that your process would be terminated if you use the *main node* for running resource demanding scripts.

To run your full analysis you will need to request a node. See https://oit.ua.edu/service/research-getting-started/ for details. An example is provided below:

`srun --mem-per-cpu 20G -q main -n 1 -c 10 --pty /bin/bash`

The parameters of the above comment should be adjusted according to your analysis. The above configuration requests 20G per CPU & 10 CPUs (cores).

**Check status of work:**
`squeue -u username`

**Using SPM12**

To use SPM12 copy the entire SPM12 folder, including the toolboxes under the toolbox folder, under the `$HOME/Software/SPM12` folder (or any other folder under your `$HOME`).

The `userpath` folder (where MATLAB looks for scripts to execute on startup) is under `$HOME/Documents/MATLAB` . I created a `startup.m` file under that folder, which includes the addpath command.To complete these steps enter the following commands:

`cd $HOME/Documents/MATLAB`
`touch startup.m`
Edit `startup.m` with a text editor (*vi* & *nano* are already installed)
`vi startup.m`
Insert:
`addpath /home/fsoylu/Software/spm12`

Now, SPM and its toolboxes should work when you run MATLAB.

## How to Create a SBATCH file

\[We need content here\]

## Tips Specific to Conducting GMV Analysis

I generated MATLAB scripts for the analysis, after creating batch files using the SPM GUI & exporting them as scripts.

I converted the paths to the T1 image files in `02_LongSegmentation_job.m` to relative paths. In general it is a good idea to use relative paths in the scripts (provided that the location of data files won't change relative to the location of scripts).

### Difference across `&`, `disown`, and `nohup`

`&` puts the job in the background, that is, makes it block on attempting to read input, and makes the shell not wait for its completion.

`disown` removes the process from the shell's job control, but it still leaves it connected to the terminal. One of the results is that the shell won't send it a SIGHUP. Obviously, it can only be applied to background jobs, because you cannot enter it when a foreground job is running.

`nohup` disconnects the process from the terminal, redirects its output to nohup.out and shields it from SIGHUP. One of the effects (the naming one) is that the process won't receive any sent SIGHUP. It is completely independent from job control and could in principle be used also for foreground jobs (although that's not very useful).

For the unfamiliar, the above commands are helpful when you want to run your script in the background and/or close the terminal in the local computer and let the script still run (`nohup` does that).

## MISC

Some notes from the UAHPC workshop on 04/03/2020:

use `module load/unload` instead of `use`

`module avail compilers`

`SBATCH -p main` #-p is for partition, main mean all of them
`SBATCH --qos main` # qos gives you 24 hours to run it
`SBATCH -e errors.%A` #%A is replaced by job ID
`SBATCH -o output.%A` #%A is replaced by job ID
`SBATCH -n 32` #allowed to run 32 cores

By default each core gets 1GB

# Basic Commands to Keep Track of SLURM Jobs
`sacct -u username --format=JobID,JobName,State,Start,End` #show all jobs submitted, with timestamps
`squeue -u username` #show currently running jobs
