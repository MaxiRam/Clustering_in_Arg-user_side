# Installing LAMMPS with the REANN pairs style for CPU only at Serafin

Step-by-step installation of lammps in Serafin with reann pair_style.
Following the guidelines from reann developer at: https://github.com/zhangylch/REANN/blob/main/manual/REANNPackage_manumal_v_1.0.pdf

I'm assuming REANN package is already installed for trainnig the models and the source code is located at $HOME/Software/REANN

Create folder and get the lammps version specified in reann manual

    export LAMMPS=$HOME/Software/LAMMPS
    mkdir -p $LAMMPS
    cd $LAMMPS
    curl https://download.lammps.org/tars/lammps-10Feb2021.tar.gz -o lammps-10Feb2021.tar.gz
    tar -xzvf lammps-10Feb2021.tar.gz
    export LAMMPS_ROOT=$LAMMPS/lammps-10Feb21
    cd
    
Create folder and get libtorch C++ cpu_only version (Serafin does not have GPU's)

    export TORCH_ROOT=$HOME/Software/libtorch
    mkdir mkdir -p $TORCH_ROOT
    cd $TORCH_ROOT
    curl https://download.pytorch.org/libtorch/cpu/libtorch-shared-with-deps-1.11.0%2Bcpu.zip -o libtorch-1.11_cpu.zip
    unzip libtorch-1.11_cpu.zip
    cd
    
Copy necessary file to the LAMMPS folder

    mv $TORCH_ROOT/libtorch $LAMMPS_ROOT
    export REANN_INTERFACE=$HOME/Software/REANN/reann/lammps-REANN-interface
    mkdir $LAMMPS_ROOT/build
    cp $REANN_INTERFACE/build/build.sh $LAMMPS_ROOT/build
    cp $REANN_INTERFACE/cmake/CMakeLists.txt $LAMMPS_ROOT/cmake
    mkdir $LAMMPS_ROOT/examples/reann
    cp $REANN_INTERFACE/examples/* $LAMMPS_ROOT/examples/reann
    cp $REANN_INTERFACE/src/* $LAMMPS_ROOT/src
    
Load cmake, compiler and library modules

    module load aocc/3.1.0 cmake/3.21.3
    module load amdblis/3.0 amdlibflame/3.0 amdlibm/3.0 openmpi/4.1.2rc1
    module load amdfftw/3.0 amdscalapack/3.0
    
Build lammps configuration

    cd $LAMMPS_ROOT/build
    cmake -D BUILD_MPI=ON -D PKG_USER-OMP=ON -D BUILD_OMP=ON -D LAMMPS_MACHINE=mpi -D CMAKE_BUILD_TYPE=RELEASE -D WITH_PNG=no -D WITH_FFMPEG=no -D FFT=FFTW3 -D FFTW3_INCLUDE_DIR=$AMDFFTW_ROOT/include -D FFTW3_LIBRARY=$AMDFFTW_ROOT/lib ../cmake
    
    
It will write some output summarizing the configuration options

```-- <<< Build configuration >>>
   Operating System: Linux
   Build type:       RELEASE
   Install path:     /home/mramos/.local
   Generator:        Unix Makefiles using /usr/bin/gmake
-- Enabled packages: USER-OMP
-- <<< Compilers and Flags: >>>
-- C++ Compiler:     /opt/ccad/21.11/software/linux-rocky8-zen/gcc-8.4.1/aocc-3.1.0-2xhzkj2bcwjzxqhifkxq6lk4pj7fpcut/bin/clang++
      Type:          Clang
      Version:       12.0.0
      C++ Flags:    -O3 -march=znver2 -mfma -mavx2 -fvectorize -O3 -DNDEBUG
      Defines:       LAMMPS_SMALLBIG;LAMMPS_MEMALIGN=64;LAMMPS_OMP_COMPAT=4;LAMMPS_PNG;LAMMPS_GZIP;LMP_USER_OMP
-- <<< Linker flags: >>>
-- Executable name:  lmp_mpi
-- Static library flags:    
-- <<< MPI flags >>>
-- MPI_defines:      MPICH_SKIP_MPICXX;OMPI_SKIP_MPICXX;_MPICC_H
-- MPI includes:     /opt/ccad/21.11/software/linux-rocky8-zen2/aocc-3.1.0/openmpi-4.1.2rc1-4erxek6vrwzwqxcl6ois2vib3j4ycglv/include
-- MPI libraries:    /opt/xpmem/lib/libxpmem.so;/opt/ccad/21.11/software/linux-rocky8-zen2/aocc-3.1.0/openmpi-4.1.2rc1-4erxek6vrwzwqxcl6ois2vib3j4ycglv/lib/libmpi.so;
-- Configuring done
-- Generating done
-- Build files have been written to: /home/mramos/Software/LAMMPS/lammps-10Feb21/build
```

Now it is time to compile and link LAMMPS

    make -j 8
    
This will take a while and write a lot of output. Go get some coffe or whatever you like...

Now `lmp_mpi` should exist in $LAMMPS_ROOT



    
