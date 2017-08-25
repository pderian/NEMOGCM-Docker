README for NEMOGCM + Docker

Pierre DERIAN - 2016-2017
pierre.derian@inria.fr | contact@pierrederian.net
www.pierrederian.net
----------------

- Purpose: Docker container to compile and run the NEMO ocean engine with parallel processing (MPI).

- Requirements: Docker and Subversion (to download NEMO source files)

With this framework, the source files are kept on the host machine.
In particular, they can be edited here using the usual tools (IDE, etc.). 
The input and output simulation files are also synchronized between the host
and the container.
The container is only used to (i) compile and (ii) run the NEMO engine. 

Tested successfully on Mac OS X >= 10.11.5
with Docker version >= 1.12.0-rc2, build 906eacd, experimental

References:
[1] http://forge.ipsl.jussieu.fr/nemo/wiki/Users
[2] http://forge.ipsl.jussieu.fr/nemo/wiki/Users/ModelInterfacing/InputsOutputs#ExtractingandinstallingXIOS
[3] http://forge.ipsl.jussieu.fr/nemo/wiki/Users/ModelInstall
[4] https://docs.docker.com/docker-for-mac/troubleshoot/#known-issues

-----------
0. Preamble
The following assumes that the current directory is the root project directory (e.g. NEMOGCM-Docker or NEMOGCM-Docker-master).
It contains:
    - this "readme.txt" file;
    - two subdirectories "arch_XIOS" and "arch_NEMOGCM" with architecture files
        for XIOS and NEMOGCM compilation;
    - a "Docker" subdirectory with a Dockerfile that builds the
        compilation/execution container. 

----------------
1. Source setup on the host

a) Download the sources for NEMOCGM, XIOS 1.0 and XIOS 2.0 in a SRC subdirectory.
/!\ Note: an account is mandatory, see the NEMO website [1].
/!\ Note: with NEMO version "nemo_v3_6_STABLE", use xios-1.0 revision 703 [2].

mkdir SRC; cd SRC
svn --username 'my_login' --password 'my_password' co http://forge.ipsl.jussieu.fr/nemo/svn/branches/2015/nemo_v3_6_STABLE/NEMOGCM
svn co -r 703 http://forge.ipsl.jussieu.fr/ioserver/svn/XIOS/branchs/xios-1.0 xios-1.0
svn co -r 990 http://forge.ipsl.jussieu.fr/ioserver/svn/XIOS/trunk xios-2.0

b) Create a symlink for XIOS 1.0 or 2.0.
This should create a XIOS directory at the same level as NEMOGCM, xios-1.0
and xios-2.0.

# either
ln -s xios-1.0 XIOS #to use XIOS 1.0
# or
ln -s xios-2.0 XIOS #to use XIOS 2.0

c) Copy the architecture files.

cd ..; cp arch_NEMOGCM/arch* SRC/NEMOGCM/ARCH; cp arch_XIOS/arch* SRC/XIOS/ARCH

----------------
2. Build the Debian image.

cd Docker; docker build -t nemo/compiler .

----------------
3. Start an interactive container sharing the source files as a Volume.
This way the host SRC will be available from within the container as /SRC.
/!\ Note: /host/path/to/SRC must be the _absolute_ path to the host SRC directory 
(at least on Mac OS X). 

docker run -v /host/path/to/SRC:/SRC -t -i nemo/compiler /bin/bash

----------------
4. Compile XIOS.
This compilation takes a while, but is done once and for all.
/!\ Note: from here on, unless mentioned otherwise, all commands are executed within
the container.

cd /SRC/XIOS
./make_xios --dev --netcdf_lib netcdf4_seq --arch DEBIAN

----------------
5. Create default NEMO configurations, compile and enjoy.
/!\ Note: here, the compilation command depends on the configuration as well as the
setup. Please refer to NEMO website for more information.
In particular, the key "key_xios2" must be added if XIOS 2.0 was chosen (not recommended with nemo_v3_6_STABLE).
Here is an example with the GYRE (and xios 1.0).

cd /SRC/NEMOGCM/CONFIG
# to compile
./makenemo -v 3 –m DEBIAN –r GYRE -n MY_GYRE 
# to run
cd MYGYRE/EXP00; mpirun ./opa
# at the end of the run, output files are available in the current directory
# as well as simultaneously available on the host.

----------------
6. Additional notes

- Drifting clocks:
The clocks of docker container and host machine may not be synced if a NTP server is not available to the host [4], so that this warning may show up:
"make: warning:  Clock skew detected.  Your build may be incomplete."
When it happens, a workaround is to run on the host (see [4], "Known Issues"):
docker run --rm --privileged alpine hwclock -s

- Regarding MPI:
Docker can use several cores depending on the host machine and Docker configuration, see Docker Preferences > Advanced > CPUs.
To enable parallel processing, run NEMO with:
mpirun -np XXX ./opa
where XXX is the number of processes to launch - see also [3] (Section 5).

- Customizing NEMO:
Custom and/or modified source files should be placed under the "MY_SRC" subdirectory of the chosen configuration.
E.g. following step 5 above: /SRC/NEMOGCM/CONFIG/MY_GYRE/MY_SRC



