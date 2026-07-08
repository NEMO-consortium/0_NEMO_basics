# Tutorial 0 - Install, Compile and Run GYRE in version 5.0.2

**Estimated time:** 15 minutes 

## 1. Prerequisities
- Access to an HPC with at least 5 cores available
- A Fortran and C compiler (intel, gfortran, ftn ...) installed
- Netcdf, hdf5 and MPI packages installed
- Knowledge to submit a script to the batch scheduler of your HPC
- Knowledge on how to run a programme on multicores (at least 5) (srun, mpirun, ...)

## 2. How to compile NEMO

### 2.1 Install and compile XIOS2

Download the XIOS2 version with

`svn co http://forge.ipsl.jussieu.fr/ioserver/svn/XIOS2/trunk <YOURXIOSDIRECTORY>`

Setup your arch files (one for the environment (.env), another for the compiler (.fcm) and a last one for the path (.path) ):
- `cd arch` to see examples of what these files look like e.g. arch/arch-ifort_MESOIPSL.*.
- setup your arch files (`arch/arch-MY_COMPUTER.*`) depending of your environement, compiler and computer.
- Then you compile XIOS (cd .. to `<YOURXIOSDIRECTORY>`) referring to your set of arch files:

`./make_xios --arch MY_COMPUTER --full --prod --job 8`

cd .. back to your workdir

XIOS2 is now compiled

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

You have the option to either use your computer specific arch file or if you do have have this reference in place, you can run the auto arch file build

*Option 1:* Use an existing arch file

Add your arch file (`arch/arch-MY_COMPUTER.fcm`) under `<YOURNEMODIRECTORY>/arch` and set up the correct path for netcdf, HDF5 and XIOS (%NCDF_HOME, %HDF5_HOME and %XIOS_HOME). Examples are available in the directory `arch`.

*Option 2:* Build an auto arch
```
cd arch/
./build_arch-auto.sh --xios_prefix <YOURXIOSDIRECTORY>
cd ..
```

If you use the auto build option, `MY_COMPUTER` is `auto`.

**Compile NEMO:**
Now, you can start compiling the configuration based on the reference configuration GYRE. The new configuration is called ‘GYRE_DEMO’.
To compile 'GYRE_DEMO' run the following line (ifort_SPIRIT is the used arch file):

`./makenemo -m MY_COMPUTER -r GYRE_PISCES -n GYRE_DEMO -j 8`

Now the configuration is compiled.

## 3. How to run NEMO 5.0.2

**Go in your configuration directory**

Copy EXP00 to EXP01 and enter the EXP01 directory

```
cp cfgs/GYRE_DEMO/EXP00 cfgs/GYRE_DEMO/EXP01
cd cfgs/GYRE_DEMO/EXP01
```

**Make sure all your your output is in a single file**

By default, if you run on, for example 4 cores, your output will be split in 4. This is cumbersome for plotting. You can use the REBUILD_NEMO tool, but we suggest below an easier option for a "light" configuration like we have here for GYRE.

Open file_def_nemo.xml

and change this line:
```
    <file_definition type="multiple_file" name="@expname@_@freq@_@startdate@_@enddate@" sync_freq="10d" min_digits="4">
```
to this: 
```
    <file_definition type="one_file" name="@expname@_@freq@_@startdate@_@enddate@" sync_freq="10d" min_digits="4">
```

**Build your submition script:**

Now you have everything you need to run your regional configuration.
For this you need to build a script to run on HPC. We suggest you use 4 MPI core.
Suggestion: look at the supercomputer documentation or copy from a friend.

Here is an exemple for a simple bash-script for running NEMO with 4 cpus and XIOS with 1 cpu: 
```
#!/bin/sh
#SBATCH --ntasks=4
#SBATCH --time 0:30:00

mpirun -np 4 ./nemo
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

**Checking output and troubleshooting**

If you job has completed coorectly, the end of your ocean.output file should look like this:

```
           iom_nf90_rp0123d, file: ./GYRE_00004320_restart_0000.nc, var: ssha wr
 itten ok
                     iom_close ~~~ close file: ./GYRE_00004320_restart_0000.nc o
 k

AAAAAAAA
```

If your job encouters a problem, search for `E R R O R` in ocean.output to give you a clue as to what may be the problem.

Once your job has completed, and your model output has been generated, you can do a quick check using ncview. Here below is an ncview of, for example, the sea surface height (`sossheig`) in file `GYRE_5d_00010101_00021230_grid_T.nc`, at timestep 144:

<img width="317" height="220" alt="Screenshot 2026-06-27 at 12 04 52 AM" src="https://github.com/user-attachments/assets/7878bcbb-a949-4bc4-a078-a06f6a1b3b51" />



