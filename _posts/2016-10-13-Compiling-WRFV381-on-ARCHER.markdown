---
layout: post
title: Compiling WRF v3.8.1 on ARCHER (Cray XC system)
tags: compilation WRF archer
---

This post documents how to compile the latest release of WRF (version 3.8.1) on the ARCHER HPC service. There is already a compiled version available on archer that be accessed using the modules funciton ([see this guide](http://www.archer.ac.uk/documentation/software/wrf/index.php)), but you will need to be able to compile from the source code if you are making any modifications to the code, or if you need to compile the idealised cases, or need to compile it ith a different nesting set up. (The pre-compiled code is set up for basic nesting only).

## Compilation with the default ARCHER compiler (Cray CCE)

This is relatively straightforward as the configure script already works as expected. However, compiling with the Cray compiler (usually the default loaded compiler when you login to ARCHER) can take upwards of 6-8 hours, depending on the options selected in configure, and the load on the login or serial nodes.

### Setting up your ARCHER environment

First, check that you do actually have the Cray compiler environment loaded. You can look for `PrgEnv-cray` in the output from running the `module list` command, or you can do `echo $PE_ENV` from a login session. 

Because it takes so long to compile using the Cray compiler, you need to run the compilation task as a serial job on the serial nodes. If you attempt to run on the login nodes, the compilation process will time out well before completion. 

So we are going to prepare three things:

1. A 'pre build' script that will load the correct modules on ARCHER and set up the environment variables
2. The configure.wrf file. This is prepared using the `configure` script, so you should not need to do anything different to the [normal WRF compilation instructions](http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compilation_tutorial.php) for this part.
3. A compliation job submission (.pbs) script. This will be submitted as a serial node job to do the actual compilation bit. 

#### The pre-build script

This is a shell script (of the bash flavour) that loads the relevant modules for WRF to compile.

{% highlight bash %}

# Set the cray compiler wrappers. Cray uses the same wrappers,
# regardless of which compiler you actually have loaded, be it
# gfortran, intel, or Cray's own compiler
export CC=cc FC=ftn F77=ftn F90=ftn CXX=CC

module load cray-libsci

# If you are compiling wrf with the netcdf 4 parallel support
module load cray-netcdf-hdf5parallel
# If you are just using standard netcdf 4:
# module load cray-netcdf

module load libpng
module load jasper
module load ncl

# This is needed for some other tools that are built, note,
# it does not overwrite the Cray compiler option for the 
# main bulk of the code's compilation
module load gcc/6.1.0

# continues...
{% endhighlight %}

NetCDF-4 is the default netcdf module on the ARCHER environment, so I am assuming you want to compile it with netcdf-4 (it has extra features like supporting file compression etc...)

I attempted this with the gcc/5.x.x module, but ran into compilation errors. Using GCC v6 seemed to fix them.

Note that you may need to switch a `load` statement for a `swap` statement in some places, depending on what default modules are loaded in your ARCHER environment. See your `.profile` and `.bashrc` scripts. 

NOTE: `.profile` (in the `$HOME` directory is only loaded on the login shells. If you launch any other interactive shells (like an interactive job mode), then `.bashrc` will get loaded.

{% highlight bash %}
# ...continued

# This next bit sets the environment variables used in the WRF configure script.
# These must be set up to match the modules you have loaded above. If you haven't
# loaded the correct module, the environment variable will not work.

export FORTRAN_COMPILER_TIMER='time -p'
export J='-j -1'
# This sets compilation to take place in serial, using only one thread.

export NETCDF=$NETCDF_DIR
export HDF5=$HDF5_DIR

# large file support seems to be set by default in the configure.wrf
# file that is generated but explicitly set it anyway
export WRFIO_NCD_LARGE_FILE_SUPPORT=1
# Parallel netCDF is experimental.  Using Parallel netCDF with the
# correct Lustre striping make an enormous difference to I/O time.
# Try a striping factor like SQRT(numprocs) for best results.
# PARALLEL_NETCDF_DIR is set in cray-parallel-netcdf.
export PNETCDF=$PARALLEL_NETCDF_DIR
# use netCDF-4 compression (note that Parallel netCDF is compatible
# with netCDF-4 since November 2013)
export NETCDF4=1

# WPS
# JASPER_DIR is set in jasper.  Note that Jasper isn't required for
# WRF, only for WPS.
export JASPERINC=$JASPER_DIR/include
export JASPERLIB=$JASPER_DIR/lib

# WRF
# Choose the core in the build script
export WRF_EM_CORE=1
export WRF_NMM_CORE=0
export WRF_DA_CORE=0

# WRF-Chem (this does NOT work with shared memory parallelism, see the
# WRF-Chem User's Guide Section 2.2.3
# http://ruc.noaa.gov/wrf/WG11/Users_guide.pdf)
# Choose Chem in the build script
export WRF_CHEM=0
# start of KPP stuff
export WRF_KPP=0
# byacc is needed
{% endhighlight %}

Some things to note: 

1. You cannot compile WRF in parallel (it's a bug in the Makefile, apparrently)
2. There are some environment variables that are just set equal to some other environmnet variables, e.g. `NETCDF=$NETCDF_DIR`. This works because when you use the `module` system to load, say, netCDF, ARCHER will automatically set its own environment variables that we can use to initialise the WRF configure variables, e.g. `$NETCDF`.

### Run `configure`

For the CRAY compiler, this can be run as normal e.g. `./configure` from the `WRFV3` directory. 

### Compilation job script

At this stage, you can either request an interactive mode job on the serial nodes, and then run compile in the usual way (after running the prebuild script and the configure commands), or you can submit a serial job with the PBS job scheduling system to run when a node becomes available. If you are going down the interactive job mode, be sure to request enough walltime as the Cray compiler takes a _long_ time to compile everything. I would ask for 12 hours to be on the safe side.

If you want to submit a job to run without having to wait for an interactive-mode job, prepare the following job submission script:

{% highlight bash %}
#!/bin/bash --login

# This script needs to be qsubbed in the build directory.

#PBS -q standard
#PBS -N CrayWRF_build
#PBS -l select=serial=true:ncpus=1
#PBS -l walltime=12:00:00
#PBS -A [YOUR ACCOUNT BUDGE CODE]
#PBS -V

# Make sure any symbolic links are resolved to absolute path
export PBS_O_WORKDIR=$(readlink -f $PBS_O_WORKDIR)

# Change to the directory that the job was submitted from
cd $PBS_O_WORKDIR/WRF_build381

# The configuration has to be done already, on the login nodes, and
# the PBS -V directive (see above) gets the environment that has been
# set up.  The compilation takes about 1 hour ('make -j 1') on a
# serial node.

# However, I don't trust the -V mode, so I run the environment set
# up script again:

./pre-build.bash

# The compile step is run on the serial nodes because the compile
# takes so long, and some optimisation steps take so long that the
# /tmp directory is emptied by the system during the optimisations,
# giving 'file not found' errors.  This seems to be occurring on the
# serial nodes as well now so $TMPDIR is set; this may affect Cray
# Fortran OPEN statements for scratch files but there are none of
# these in WRF, WPS or Chem.

mkdir -p tmp
export TMPDIR=$PWD/tmp

# WRF; ARW core
(
    cd WRFV3
    ./compile em_real &> compileCray.log
)

# WPS
(
    cd WPS
    ./compile &> compileCray.log
)

unset TMPDIR
rm -rf tmp
{% endhighlight %}

### Putting it all together.

1. Make sure the `pre-build.bash` script and the `compile.pbs` script are in a directory above `/WRFV3`. I called it `WRF_build381`

2. Use qsub as normal to submit the compile.pbs script. E.g. `qsub compile.pbs`

Your job should run and the compile logs will be written to compileCray.log (or whatever you named them in the `compile.log` script above.

## Compiling WRF using the GNU compilers on ARCHER

You may have reason to want to compile WRF with the GNU compilers on ARCHER or another Cray XC30 system. Unfortunately I found that the `configure` script supplied with version 3.8.1 did not generate a correct `configure.wrf` script for the GNU compilers in a Cray enviroment. Namely, it used compilation flags specific to the Cray compiler, rather than the gfortran compilation flags (which are incompatible). To rectify this you can either run the `configure` script as normal, and then correct the compiler flags in the `configure.wrf` output script that is generated. Or if you want a more re-usable soultion you can edit the file in the `WRFV3/arch/configure_new.defaults` file.

I did this by opening the `configure_new.defaults` file and adding a new entry. The purpose of the file is to generate the menu entries that you see when running the `configure` script, and then populate the Makefile with the correct compilation options. 

Find the CRAY CCE entry in the `configure_new.defaults` file and insert a new entry below it called `GNU on CRAY XC30 system` or similar. The entry should contain the following:

```
###########################################################
#ARCH    Cray XE and XC CLE/Linux x86_64, GNU Compiler on Cray System # serial dmpar smpar dm+sm
# Use this when you are using the GNU programming environment on ARCHER (a Cray system)

DESCRIPTION     =       GNU on Cray system ($SFC/$SCC): Cray XE and XC
# OpenMP is enabled by default for Cray CCE compiler
# This turns it off
DMPARALLEL      =       # 1
OMPCPP          =       # -D_OPENMP
OMP             =       # -fopenmp
OMPCC           =       # -fopenmp
SFC             =       ftn
SCC             =       cc
CCOMP           =       gcc
DM_FC           =       ftn
DM_CC           =       cc
FC              =       CONFIGURE_FC
CC              =       CONFIGURE_CC
LD              =       $(FC)
RWORDSIZE       =       CONFIGURE_RWORDSIZE
PROMOTION       =       #-fdefault-real-8
ARCH_LOCAL      =       -DNONSTANDARD_SYSTEM_SUBR  -DWRF_USE_CLM
CFLAGS_LOCAL    =       -O3
LDFLAGS_LOCAL   =
CPLUSPLUSLIB    =
ESMF_LDFLAG     =       $(CPLUSPLUSLIB)

FCOPTIM         =       -O2 -ftree-vectorize -funroll-loops
FCREDUCEDOPT    =       $(FCOPTIM)
FCNOOPT         =       -O0
FCDEBUG         =       # -g $(FCNOOPT) # -ggdb -fbacktrace -fcheck=bounds,do,mem,pointer -ffpe-trap=invalid,zero,overflow
FORMAT_FIXED    =       -ffixed-form
FORMAT_FREE     =       -ffree-form -ffree-line-length-none
FCSUFFIX        =
BYTESWAPIO      =       -fconvert=big-endian -frecord-marker=4
FCBASEOPTS_NO_G =       -w $(FORMAT_FREE) $(BYTESWAPIO)
FCBASEOPTS      =       $(FCBASEOPTS_NO_G) $(FCDEBUG)
FCBASEOPTS_NO_G =       -N1023 $(FORMAT_FREE) $(BYTESWAPIO) #-ra
FCBASEOPTS      =       $(FCBASEOPTS_NO_G) $(FCDEBUG)
MODULE_SRCH_FLAG =
TRADFLAG        =      -traditional
CPP             =      /lib/cpp -P
AR              =      ar
ARFLAGS         =      ru
M4              =      m4 -G
RANLIB          =      ranlib
RLFLAGS         =
CC_TOOLS        =      gcc


###########################################################
```

You can run the configure script as normal once these changes have been made and you will get a `configure.wrf` suitable for using the GNU compilers on ARCHER to build WRF v3.8.1.






