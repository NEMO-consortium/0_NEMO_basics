# Tutorial 0 - Install, Compile and Run GYRE in version 5.0.2 with XIOS3

## 1. Prerequisities
- Access to an HPC with at least 5 cores available
- A Fortran and C compiler (intel, gfortran, ftn ...) installed
- Netcdf, hdf5 and MPI packages installed
- Knowledge to submit a script to the batch scheduler of your HPC
- Knowledge on how to run a programme on multicores (at least 5) (srun, mpirun, ...)

## 2. How to compile NEMO

### 2.1 Install and compile XIOS3

Download the XIOS3 version with

`svn co http://forge.ipsl.jussieu.fr/ioserver/svn/XIOS3/trunk <YOURXIOSDIRECTORY>`

Setup your arch files (one for the environment (.env), another for the compiler (.fcm) and a last one for the path (.path) ):
- cd arch to see examples of what this file looks like e.g. arch/arch-ifort_MESOIPSL.*.
- setup your arch files (`arch/arch-MY_COMPUTER.*`) depending of your environement, compiler and computer.
- Then you compile XIOS (cd .. to `<YOURXIOSDIRECTORY>`) referring to your set of arch files:

`./make_xios --arch MY_COMPUTER --full --prod --job 8`

cd .. back to your workdir

XIOS3 is now compiled

### 2.2 Install and compile NEMO version 5.0.2

**Download NEMO version 5.0.2:**

`git clone --branch 5.0.2 https://forge.nemo-ocean.eu/nemo/nemo.git <YOURNEMODIRECTORY>`

and go in:

`cd <YOURNEMODIRECTORY>`

**Build your arch file**
load the environement (similar to the one used for XIOS):
```
module load netcdf-fortran netcdf-c hdf5 mpi fortran_compiler
```
The exact name and version of each module is computer dependent. To find out what is already installed, you can run `module avail` and then `module load <YOUR_MODULES>`.

**Build the arch file:**
```
cd arch/
./build_arch-auto.sh --xios_prefix <YOURXIOSDIRECTORY>
```

After successfully downloading, add your arch file (`arch/arch-MY_COMPUTER.fcm`) under `<YOURNEMODIRECTORY>/arch` and set up the correct path for netcdf, HDF5 and XIOS (%NCDF_HOME, %HDF5_HOME and %XIOS_HOME). Examples are available in the directory `arch`

**Compile NEMO:**
Now, you can start compiling the configuration based on the reference configuration GYRE, as we use XIOS3 the keys in the compilation need to be changed. The new configuration is called ‘MY_GYRE’.
To compile 'MY_GYRE' run the following line (ifort_SPIRIT is the used arch file):

`./makenemo -m MY_COMPUTER -r GYRE_PISCES -n MY_GYRE -j 8 --add_key key_xios3`

Now the configuration is compiled.

## 3. How to run NEMO 5.0.2

**Go in your configuration directory**
```
cd cfgs/MY_GYRE/EXP00
```

**Update your iodef.xml for XIOS3:**

For the use of XIOS3, NEMO and XIOS3 needs to be run in detached mode.
This means, in the file` iodef.xml` the following line needs to be:  
` <variable id="using_server"              type="bool">true</variable>`  
and the following line needs to be removed or commented:  
` <variable id="oasis_codes_id"            type="string" >oceanx</variable>`  

Last thing to do, is to copy the `xios_server.exe` into the folder of the configuration:    
` cp <YOURXIOSDIRECTORY>/bin/xios_server.exe ./ `

**Build your submition script:**

Now you have everything you need to run your regional configuration.
For this you need to build a script to run on HPC. We suggest you use 4 MPI core.
Suggestion: look at the supercomputer documentation or copy from a friend.

Here is an exemple for a simple bash-script for running NEMO with 4 cpus and XIOS with 1 cpu: 
```
#!/bin/sh
#SBATCH --ntasks=5
#SBATCH --time 0:30:00

mpirun -np 4 ./nemo -np 1 ./xios_server.exe
```

The command `mpirun` is HPC dependent and can vary between machines. On some HPC, you may need to load your modules used to compile NEMO in the script itself by adding before starting NEMO:
```
module purge
module load <YOUR_MODULES>
```

**Run NEMO:**

Now you can submit your job to run the simulation (command is computer dependent).

Once terminated, you now have run your first NEMO simulation :).

This is the simplest way to run NEMO with XIOS3.
