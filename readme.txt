# :package: :key: AMCAFE_DSTG
## :pencil: Description
This repo contains the modifications to the [AMCAFE](https://github.com/USNavalResearchLaboratory/AMCAFE.git) original created by Teferra Kirubel (Please see the following [publication](https://doi.org/10.1016/j.actamat.2021.116930).

Included below are a description of the tool from the original AMCAFE repo and the installation instructions. The locations of this information is provided under *AMCAFE Original*.

## :mushroom: New updates
The updates to the code that are included in this repo are the updates to the scan strategies. We have been interested in mult-spot scan strategies. These included random, and two different skip strategies proposed by [Arold et al](https://doi.org/10.1016/j.jmapro.2023.10.070).

### Random scan strategies
For the random scan strategies, were interested in investigating how randomness could be manipulated to alter the microstructure. We did this by creating a repeatable pseudo-random algorithm to shuffle a unidirectional scan strategy. The level of random was dictated by the value of $n_{ord}$. As the values of $n_{ord}$ decreases, the random shuffling increases. To visualise the developed algorithm, a Jupyter Document in the Visualisation/ folder can be used. The algorithm is provided in the python file.

### Alternate scan strategies
The fully constant and sequentially constant scan strategies proposed by [Arold et al](https://doi.org/10.1016/j.jmapro.2023.10.070) can also be implemented. In this work, $d_{jump}$ is specified as multiples of the nsd value, which is the number of point in the scan direction. The number of points is determined by the input values:
$$\frac{gridsize[0]}{bmv*bmDelT} + 1$$

where the $bmv$ is the beam velocity (in m/s), gridsize should be a value close to $1.6e-3$, and bmDelT is the time step for simulating the moving beam as defined by [Schwalback et al](https://doi.org/10.1016/j.addma.2018.12.004). This should be taken as $bmDelT=\frac{4\sigma}{3bvm}$ where $\sigma$ is the beam radius.

Therefore, specifying a $d_{jump}$ of 2, leads to 2*nsd. This means that given the input parameters you are supplying, you should also calculate nsd to determine the extent of beam overlap needed with the $d_{jump}$ you decide to use.

For the fully constant scan strategy, the $d_{jump}$ remains constant throughout the layer and between the layer; however, for the sequentially constant, the $d_{jump}$ changes between layers. This means that you must supply a lower and upper bound for $d_{jump}$.

So lets see how you would do this in the input file used to run AMCAFE in the next section.

## :inbox_tray: Input File Features
Example input files can be found in the *ExampleInputFiles/* folder. The following sections detail the important inputs.

### Scan strategy selection
To specify which scan strategy you are interested in using, you must use a value from 1 to 7 after the word *patternID* :
```bash
patternID 1
```
The following provides information on what each number represents in terms of scan strategy:
1 & 3: scan in +X direction only
2 & 4: scan in alternating +/- X direction
5: fully constant
6: sequentially constant
7: pseudo random

### Additional information for each scan strategy
If *patternID* 6 or 7 is used, you must also specify the level of randomness you would like to introduce. This is done using *nord*:

```bash
nord 0.1
```
Depending on how random you would like the selection to be, you can select any value between 0 and 1, where close to 0 is extreme random and close to 1 is limited random.

If you would like them random or sequentially constant values to change with each layer, you must specify this using either a value of 0 or 1, where 0 is *no* and 1 is *yes*.

```bash
layerchange 1
```

For sequentially constant scan strategies, the $d_{jump}$ needs to change with each layer. To achieve this an upper and lower bound for $d_{jump}$ need to be specified:

```bash
djumpl 2
djumpu 3
```

If fully constant is to be used, just one value for $d_{jump}$ is required:

```bash
djump 2
```

To improve the computational effeciency of the model, you can also chose to create the baseplate grains outside the program and have them read in. If you prefer to do this, you can specify this using the input file:

```bash
vtkread 1
```

You must also provide two files, the vtk file of the baseplate microstructure and the featuredata from which the orientation data is read from. This is specified in the command line and therefore should be located with the input file:

```bash
mpirun -np 1 CAFE example.in vtk.vtk featuredata.csv
```

In the *data_dream3d* folder, an example pipeline is provided *baseplate.json* which you can use to generate the necessary data.

Examples for all three possibilities are provided in the *EamplesInputFiles* folder.

## Data processing
Three files original developed by Teferra Kirubel are also included. These files with another file *grain_info.py* are used to alter the files created by AMCAFE for visualisation, as well as provide means to manipulate and analyse the formed data.

Examples of how to use these files is provided in the *Jupyter Document* in the same folder.

### DREAM3D Files
In the folder *data_dream3d* there are pipelines which can be used to import *vtk* versions of the microstructures manufactured using AMCAFE and generate grain information such as distance to boundaries which can be used by other funtions.

### MTEX
In the folder MTEX, there are files in there which will take the odf text files generated by Python (shown in the *DataConversion&Visualisation* Jupyter notebook) and generate IPF plots (*Faster_PlotIPFFCC_MTEX5_4.m*), misorientation density plots (*misorientation_FCC_MTEX5_4.m*), and the pole figures (*Faster_PlotPoleFCC_MTEX5_4.m*).

## Dylan installation notes

This tool is a Linux specific software.  The required software for installation includes:
* Openmpi
* Cmake >= 3.14
* HDF5-parallel
* Adios2 
* Metis
* GKlib

The following installation instructions assume you have openmpi and cmake installed.

### HDF5-parallel
* The current version which has been tested and used is hdf5-1.12.0 
* Unzip and cd into folder
* Configure for installation: ./configure --enable-parallel --enable-shared
* make
* make install

### Adios2
Adios2 is relatively easier to install, the following steps are required:

* Firstly, download from the git respository found [here](https://github.com/ornladios/ADIOS2.git)
* Extract the zipped folder and cd into it and create a new folder called *build*.
* Now run the following *cmake* command where *-DCMAKE_INSTALL_PREFIX* points to the newly download folder (which will be the location of the installation)
, and *-DHDF5_ROOT* is the location where HDF5 is located: cmake -DCMAKE_INSTALL_PREFIX=/usr/local/ADIOS2 -DADIOS2_USE_Fortran=OFF -DADIOS2_USE_MPI=ON -DADIOS2_USE_HDF5=ON -DHDF5_ROOT=/usr/local/hdf5-1.12.0/ ../../ADIOS2
* Once configured you can make, make install.

### Metis and GKlib
* Download GKlib from the following [location](https://github.com/KarypisLab/GKlib.git)
* Extract the zipped folder.
* Configure make file: *make config prefix=/path/to/install location*
* Where *prefix* points to the location where GKlib is to be installed.
* Download METIS from the following [location](https://github.com/KarypisLab/METIS.git)
* Extract the zipped folder
* Now lets configure the make file : *make config shared=1 gklib_path=/path/to/gklib prefix=/path/to/install
* Where *gklib_path* is the path to the GKlib folder and *prefix* is the path to where you want METIS to be installed.
* Once configured you can make, make install

### gsl
* Download version 2.2.7 from the following [location](https://www.gnu.org/software/software.html#HowToGetSoftware)
* Extract the zipped folder and cd into the folder
* Run *./configure --prefix=/path/to/installation/directory*
* Then *make*
* Then *make install*


### AMCAFE Installation
With all the dependencies installed, we can now install AMCAFE:
* We need to update the make file which can be found in the *src* folder. 
* You can follow the make file example named *Makefile_example*
* Firstly, cd into the ..ADIOS2/bin directory to determine the flags and library locations of the ADIOS2 compile using the following commands:

./adios2-config --cxx-flags -> add this information to the AMCAFE make file at *-CPPFLAGS=* (It should look like: -DADIOS2_USE_MPI -isystem /usr/local/ADIOS2/include)

./adios2-config --cxx-libs -> add this information to the AMCAFE make file at  *adioslib=*

* Update the location of the Metis library in the AMCAFE make file at *METISLIB=* (It should something like this: -L/usr/local/metis/lib -lmetis)
* Update the location of the GSL library at *GSLLIB=* (-L/usr/local/gsl-2.7/lib -lgsl -lgslcblas -lm)
* Update the location of the GK library at *GKLIB=* (-L/usr/local/GKlib/lib -lGKlib)
* Update the location of the Metis, GKlib and GSL include file location at *CPPINCLUDE=* (which could look something like: -I/usr/local/metis/include -I/usr/local/gsl-2.7/include -I/usr/local/GKlib/include)
* Update the location of mpic++ at *CPP=* (which could look like: /sw/software/OpenMPI/4.0.3-GCC-9.3.0/bin/mpic++)


## :old_key: AMCAFE Original
### 1. Description
This is an implementation of the Cellular Automata Finite Element (CAFE) algorithm to simulate solidification of additively manufactured metals. Description of the code can be found in

Teferra, Kirubel, and David J. Rowenhorst. "Optimizing the cellular automata finite element model for additive manufacturing to simulate large microstructures." Acta Materialia 213 (2021): 116930

Simulation results in the paper can reproduced using the input files in the examples directory. Below contains compilation instructions for various systems. These instructions mostly serve as guidelines to help compile on your own system. In addition to compilation instructions, the src/ directoriescontain the makefiles used to compile the code.

### 2. Compiling and running the AMCAFE code

The personal notes below show compilation notes for 3 systems titled neocortex, gaffney, and onyx (gaffney and onyx are DoD HPC machines, neocortex is ubuntu OS). The code is a C++ code that requires an MPI wrapper as well as external packages ADIOS2 and metis. The notes below give some guidance on how ADIOS2 can be compiled. METIS is a straightforward compilation. A static metis library can easily be compiled following instructions on the website. The variables in makefile in this folder need to specify the paths to the required libraries, executables, and include files. These must be adjusted to your system. After which, the executable can be built by typing "make cafe" in the terminal. The code is run by

mpirun -n <number_of_processors> ./cafe <input_file_name>

The executation command may need minor adjustments depending on your compiler. The *sh files in the folder give examples of how to run. The examples in paper referenced above can be run by input file SDX1_.in (example 1) and SDXY1_.in (example 2)


For questions on compiling and running, please email: kirubel.teferra@nrl.navy.mil

#### 2.1 compiling ADIOS2 to be able to output HDF5

dependencies: HDF5 parallel and cmake

neocortex:
HDF5
1) untarred hdf5 and cd'd to directory /usr/local
2) chmod 755 for the directory
3) did the following command
CC=mpicc ./configure --enable-parallel
make -j 12
make install

* note that mpicc refers to a petsc build already on neocortex:
/opt/petsc/arch-linux2-c-opt/bin/mpicc

CMAKE

option 1

1) downloaded cmake source and untarred, cd'd to directory
2) needed the openssl library (had an error first)
sudo apt get install libssl-dev
3) then built cmake as
./bootstrap
sudo make -j 12
make install

the did another chmod -R 755 . for the entire directory

option 2

download the cmake-version-.sh online then do
sudo sh cmake-$version.$build-Linux-x86_64.sh --prefix=/opt/cmake
then you can put the path with the cmake executable in PATH or just give full path when use cmake. Then, not sure if necessary but I make cmake RWX for everyone

ADIOS2

I am using this cmake: /opt/cmake/cmake-3.19.0-Linux-x86_64/bin/cmake
also, i make sure i'm using this mpicc: /opt/petsc/arch-linux2-c-opt/bin/mpicc, by specifying that directory in my PATH variable in ~/.profile

I created a folder /usr/local/ADIOS2 with sudo then gave 777 permissions

1) git cloned it then created subdirectory adios2-build and cd'ed to it
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/ADIOS2 -DADIOS2_USE_Fortran=OFF -DADIOS2_USE_MPI=ON -DADIOS2_USE_HDF5=ON -DHDF5_ROOT=/usr/local/hdf5-1.12.0/ ../../ADIOS2
2) make -j 12
3) make install

Then had to again give 777 permissions in all created folders

The adios-config file is in the ../ADIOS/bin directory; to determine flags to compile your application with
4) ./adios-config --cxx-flags
5) ./adios-config --cxx-libs

#### 2.2 compiling gsl

1. download gsl from https://www.gnu.org/software/gsl/doc/html/ (presently using version 2.7)
2. untar and follow the instructions in the file named "INSTALL" in the untarred directory

#### gaffney:

already had a module for cmake and HDF5 parallel so needed to load those modules,
1) module load Cmake/3.18.1
2) module load hdf5-parallel/intel-18.1.163/1.10.5
3) module load compiler/intelmpi/2019.5.281 
4) module load gcc/9.2.0 (necessary to provide paths for cmake)
5) compile ADIOS2
5a) make directory for ADIOS2, git cloned to directory, mkdir adios2-build and cd'ed to it
5b) cmake -DCMAKE_INSTALL_PREFIX=/p/home/kteferra/Documents/software/ADIOS2/ -DADIOS2_USE_MPI=ON -DADIOS2_USE_HDF5=ON -DHDF5_ROOT=/app/hdf5-parallel/1.10.5-intel-2018.1.163-intelmpi -DADIOS2_USE_Fortran=NO ../../ADIOS
(notice that it only works if I turn off fortran)
5c) Make -j 16
5d) Make install
The adios-config file is in the ../ADIOS/bin directory; to determine flags to compile your application with
5e) ./adios-config --cxx-flags
5f) ./adios-config --cxx-libs


lastly when compile (AND EXECUTE) an application that uses adios2, you have to make sure you have the following modules loaded
module swap compiler/intel/2019.4.243 compiler/intel/2019.5.281
module swap mpt/2.20 compiler/intelmpi/2019.5.281
module load hdf5-parallel/intel-18.1.163/1.10.5
module load gcc/9.2.0

#### ONYX:

As onyx is a CRAY system ADIOS2 compiled as a static library. Here are the steps:
1) load appropriate system modules
1a) module swap PrgEnv-cray PrgEnv-intel/6.0.5
1b) module load intel/19.0.1.144
1c) module load cray-hdf5-parallel/1.10.5.0
1d) module load gcc/8.3.0

2)cmake:
2a) downloaded and untared cmake-3.18 then did
2b) ./bootstrap
2c) make -j 12
2d) make install

3) ADIOS2
3a) created ADIOS2 directroy, git cloned ADIOS2 to directory, created subdir adios2-build and cc'ed to it
3b) ../../cmake-3.18.2/bin/cmake -DCMAKE_INSTALL_PREFIX=/p/home/kteferra/Documents/software/ADIOS2 -DADIOS2_USE_Fortran=OFF -DADIOS2_USE_MPI=ON -DADIOS2_USE_HDF5=ON ../../ADIOS2
3c) make -j 12
3d) make install

you can find compiler flags by running executable adios2-config in ADIOS2/bin

20200526:
if system does not have MPI wrapper, you can download petsc tar file, untar and install with:
./configure --with-cc=gcc --with-cxx=g++ --with-debugging=0 COPTFLAGS='-O3 -march=native -mtune=native' 
CXXOPTFLAGS='-O3 -march=native -mtune=native' FOPTFLAGS='-O3 -march=native -mtune=native'
 --download-mpich --download-metis


On DoD HPC system: better to use intel compiler than gcc and use as many existing modules as possible as they are optimized for system



