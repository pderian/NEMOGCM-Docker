### Installs the framework for the NEMO Ocean Engine compilation ###
####################################################################
FROM debian:jessie
MAINTAINER Pierre Derian, contact@pierrederian.net, www.pierrederian.net

### dependencies for NEMO:
# bash, perl, svn, fortran90 compiler, mpi, netcdf, netcdf-fortran, make
# from http://www.nemo-ocean.eu/Using-NEMO/User-Guides/Basics/NEMO-Quick-Start-Guide
### dependencies for XIOS: netcdf4 hdf5
# from http://forge.ipsl.jussieu.fr/ioserver/wiki/documentation
### notes:
# ssh is required in order for MPI to run properly
# nano, python2.7 are not mandatory but useful when we have to work in the container
ENV PACKAGES="subversion make gcc g++ ssh" \
    PACKAGES_NEMO="perl gfortran libnetcdf-dev libnetcdff5" \
    PACKAGES_XIOS="liburi-perl libopenmpi-dev openmpi-bin libhdf5-openmpi-dev" \
    PACKAGES_USER="nano python2.7"

### first update then install the main packages
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends ${PACKAGES} ${PACKAGES_NEMO} ${PACKAGES_XIOS} ${PACKAGES_USER}
