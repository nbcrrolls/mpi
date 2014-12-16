.. hightlight:: rst

MPI Roll
================

.. contents::

Introduction
--------------
This roll is an adaptation of SDSC's  mpi-roll. 
Cloned on 2014-12-16,  output of ``git describe``: ::

    v6.2-19-gf3122ae

To make a roll with openmpi (no mpvarich2) for eth and mx fabric and 
gnu and intel compilers use the following command: ::

    make ROLLCOMPILER='gnu intel' ROLLMPI='openmpi'  ROLLNETWORK='eth mx' 2>&1 | tee build.log

Downloads
-----------
Roll installs openmpi v.1.8.3 and knem v.1.1.1: ::

    wget http://www.open-mpi.org/software/ompi/v1.8/downloads/openmpi-1.8.3.tar.gz
    wget http://gforge.inria.fr/frs/download.php/33422/knem-1.1.1.tar.gz


Changes
---------
::

    Update modules install path to /opt/modules/mpi/[gnu,intel] (from .gnu .intel)
    Update roll test with correct module path 


Original sdsc-roll README
----------------------------------
Info below is from the original roll README

**Overview**

This roll bundles various flavors of the MPI library.
For more information about the various packages included in the mpi roll please visit their official web pages:

- `MVAPICH2 <http://mvapich.cse.ohio-state.edu/overview/mvapich2/>`_
   is an MPI-3 implementation.
- `MPI <http://www.open-mpi.org>`_ is an open source MPI-2 implementation that 
   is developed and maintained by a consortium of academic, research, and industry partners.

**Requirements**

To build/install this roll you must have root access to a Rocks development
machine (e.g., a frontend or development appliance).
Download the appropriate mpi source file(s) into the `src/<package>`.

**Dependencies**

- autoconf >= 2.69 (mvapich2).  You can get this from the SDSC gnutools-roll.
- The sdsc-roll must be installed on the build machine, since the build process
  depends on make include files provided by that roll.
  The roll sources assume that modulefiles provided by SDSC compiler
  rolls are available, but it will build without them as long as the environment
  variables they provide are otherwise defined.

**Building**

To build the mpi-roll, execute this on a Rocks development
machine (e.g., a frontend or development appliance): ::

    # make 2>&1 | tee build.log

A successful build will create the file ``mpi-*.disk1.iso``.  

This roll source supports building with different compilers.  The
``ROLLCOMPILER`` make variable can be used to specify the names of compiler
modulefiles to use for building the software, e.g.,  ::

    # make ROLLCOMPILER='gnu intel' 2>&1 | tee build.log

The build processes recognizes the values ``gnu``, ``intel`` and ``pgi`` for the
ROLLCOMPILER value, defaulting to ``gnu``.

By default, the roll builds both openmpi and mvapich2 rpms.  You can limit the
build to one or the other using the ROLLMPI make variable, e.g., ::

    # make ROLLMPI='mvapich2' 2>&1 | tee build.log

By default, the roll builds for ethernet network fabric.  You can expand this
by specifying one or more of the values ``ib`` and ``mx`` in the ROLLNETWORK make
varible, e.g., ::

    # make ROLLNETWORK='ib' 2>&1 | tee build.log

For gnu compilers, the roll also supports a ``ROLLOPTS`` make variable value of
``avx`` indicating that the target architecture supports AVX instructions.
If ``ROLLOPTS`` contains one or both of ``torque`` and ``sge`` then openmpi is built
to integrate with the specified scheduler(s).  If ``ROLLOPTS`` contains ``torus``
then mvapich2 is compiled with 3d torus support.


**Installation**

To install, execute these instructions on a Rocks frontend: ::

    # rocks add roll *.iso
    # rocks enable roll mpi
    # (cd /export/rocks/install; rocks create distro)
    # rocks run roll mpi | bash
    
In addition to the software itself, the roll installs mpi environment
module files in: ::

    /opt/modulefiles/mpi/.(compiler)

**Testing**

The mpi-roll includes a test script which can be run to verify proper
installation of the roll documentation, binaries and module files. To
run the test scripts execute the following command(s): ::

    # /root/rolltests/mpi.t 

