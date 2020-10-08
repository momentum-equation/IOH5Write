IOH5Write_b_OFv6
===============

This branch is based on a fork from https://github.com/hakostra/IOH5Write. The library of this branch ('IOH5Write_b_OFv6') provides the possibility to include patches and/or boundary data to be written to a hdf5 archive (beside the usual possibility to write internalField and kinematic cloud data to hdf5). It works with newer OpenFOAM versions. It was adapted for and tested with OpenFOAM-6. 

The functionObject library writes OpenFOAM cases as HDF5 archives instead of the default (uncollated I/O) one-file-per-process-per-timestep-per-variable approach. This saves a lot of files, makes it easier to manage, copy, and post-process the results. An XDMF file is used to describe the contents of the HDF5-file and this can easily be opened in ParaView, VisIt or any other common postprocessor tool. The I/O part is handled by MPI-IO, which makes it effective on clusters and high-performance computers with thousands of cores and parallel file systems. By writing to the hdf5 archive, no reconstruction process (reconstructPar) is necessary to reconstruct the fields and the mesh. This can save a lot of time when it is needed to reconstruct numerous time steps. Furthermore, the hdf5 archives need less disk space in comparison to OpenFOAM's uncollated and collated output. The library is compatible with the new collated file format of OpenFOAM.


Installation
------------
1. Make sure you have a working copy of OpenFOAM-6. Make sure that you have all necessary compilers and development libraries, including MPI. 
2. Install the HDF5-library. Make sure that you install or compile it with parallel/MPI support. In Ubuntu this is done by installing the package ``libhdf5-openmpi-dev``. It might be necessary to compile both OpenFOAM and HDF5 against the same MPI library and version. See the section 'hints on the hdf5-compilation' below.
3. Grab a copy of this branch, and enter the code directory. You can for example place the code in your ``~/OpenFOAM/username-6`` folder.
4. Set the environment variable ``HDF5_DIR`` to your HDF5 installation directory in the Make/options file. It makes the variable "visible" for the compile script with ``export HDF5_DIR=/path/to/hdf5-1.10.5/build/``. Do the same for ``SYSTEMOPENMPI`` in the Make/options file. Make the variable "visible" with ``export SYSTEMOPENMPI=/path/to/openmpi/include/``.
5. Compile the code with the common ``wmake libso`` and wait. This command must be executed within the directory where the Make folder is located.

If you encounter any problems during the installation, this is probably because you have a problem with either the OpenFOAM installation, HDF5 library or have failed to follow the instructions properly. You should pay special attention to the HDF5 library and make sure that the parallel support is enabled.


Some hints on the HDF5-compilation
----------------------------------
I generally build HDF5 from scratch, as the pre-build binaries available around on the different systems I use is all build with different options. Building HDF5 does not require any special libraries, but it is easy to miss that HFD5 automatically disable the building of shared libraries (which is necessary for OpenFOAM) when building parallel versions.

This branch was tested with hdf5-1.10.5. You can download it from https://portal.hdfgroup.org/display/support/Download+HDF5.

I have found the following procedure and configuration well suited.
Extract the downloaded ``hdf5-1.10.5.tar.gz`` into your preferred installation folder.
Go into the extracted ``hdf5-1.10.5`` directory.
Configure with:

``CC=mpicc CFLAGS=-fPIC LDFLAGS=-fPIC ./configure --prefix=/path/to/hdf5-1.10.5/build --enable-parallel --enable-shared``

Build hdf5 in two steps by using the commands 

``make`` 

and afterwards 

``make install``.


Choosing between single and double precision IO
-----------------------------------------------
The solution of your problem (that is going to be written) is usually found by an iterative method, that gives an approximate answer. This approximation typically has 6-8 significant digits, far from the 15 significant digits in double precision. To save space (a lot actually) it is possible to write the solution data in single precision, independent on the precision settings in OpenFOAM.

This choice of output precision is done at compile time, and the default is single precision (32 bit floating point numbers). To switch, you set either ``-DWRITE_SP`` (for single prec.) or ``-DWRITE_DP`` (for double prec.) in the ``Make/options`` file. 


Writing XDMF files
------------------
The XDMF files can be generated by using the python script ``writeXDMF_b_v2.py``. The script will, if not supplied with any additional arguments, parse the file ``h5Data/h5Data0.h5``, and write the resulting XDMF files in a folder called ``xdmf_b``. Single XDMF-files will be created for the internalField/mesh data, and/or for patch/boundary data, and/or for each cloud of particles. Usage instructions can be given with the option ``--help``.


Testing
-------
Test the code by running the inluded cavity or pitzdaily tutorial and by using ``./Allrun`` script. A directory called ``h5Data`` should be created, in which the file ``h5Data0.h5`` is created. To use the code on other cases, look in the ``controlDict`` file in this case.


Compatibility
-------------
The code works with OpenFOAM-6. There is a chance that it works also with other versions but this was not tested yet. It is recommended to use HDF5 version 1.10.5 or newer (https://portal.hdfgroup.org/display/support/Download+HDF5). The testing is only done with GCC compilers.


Known bugs and limitations
--------------------------
There are a few known bugs and limitations:

1. The code only works in parallel. This is a consequence of the design of OpenFOAM, since ``MPI_Init`` is never called for serial runs, hence the parallel HDF5 library cannot use MPI-IO.
2. With the branch 'IOH5Write_b_OFv6' it is possible to additionally write patch or boundary data to the hdf5 archive. Up to now this is only possible for quadrilateral faces. In general, it is not possible to write out boundary data for boundaries having the boundary condition "empty".
3. Like in the base work from https://github.com/hakostra/IOH5Write, the library is able to write scalar and vector fields to the hdf5 archive. Up to now, the implementation for tensor fields is still missing.
4. Like in the base work from https://github.com/hakostra/IOH5Write, the code has no clean simulation ending.
5. The code does not utilize many of the object-oriented features C++ gives. This is partly because the HDF5 library is a pure C library, requiring you to deal with pointers to arrays and so on, partly due to our lack of C++ skills.
6. The option to write the particle attribute ``Us`` (slip velocity) to a hdf5 archive is commented out at the moment since the relevant part in the file ``h5WriteCloud.C`` still needs to be adapted to newer OpenFOAM versions (OpenFOAM-6).


Found yet another bug? Got suggestions for improvements?
----------------------------------------------
Feel free to mention it in the discussion thread at [www.cfd-online.com](http://www.cfd-online.com/Forums/openfoam-programming-development/122579-hdf5-io-library-openfoam.html).

