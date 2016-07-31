# Shift
Yet another wrapper around SHIFTX+ for MD trajectories analysis but using MPI and HDF5

## Prerequisites

You need, at a minimum:

* Python 2.7
* NumPy
* Pandas
* MDAnalysis
* H5py
* Matplotlib
* MPI4py

And *SHIFTX2* chemical shift prediction tool (http://www.shiftx2.ca/).

## Installation on UNIX

I highly recommand you to install the Anaconda distribution (https://www.continuum.io/downloads) if you want a clean python environnment with nearly all the prerequisites already installed (NumPy, Pandas, Matplotlib, MPI).

In order to use hdf5 format with MPI, you need a special version compiled in parallel mode (with mpi4py).

1 . First, you need to install mpi4py
```bash
conda install mpi4py
```

2 . Now, remove hdf5 and h5py from Anaconda (trust me)
```bash
conda remove hdf5 h5py
```

3 . After, download the latest version of hdf5 and h5py
```bash
wget http://www.hdfgroup.org/ftp/HDF5/current/src/hdf5-1.8.17.tar # check if it is the latest version
git clone https://github.com/h5py/h5py.git
```

4 . Compilation of HDF5
```bash
tar -xvf hdf5-1.8.17.tar
cd hdf5-1.8.17
PREFIX=~/Applications/anaconda    # You need to adapt this to your system
./configure --prefix=$PREFIX --enable-linus-lfs --with-zlib=$PREFIX --enable-parallel --enable-shared
make
make install
```

5 . Installation of h5py
```bash
cd ../h5py
export CC=mpicc
python setup.py configure --mpi --hdf5=~/Applications/anaconda # Again, you have to adapt this
python setup.py build
python setup.py install
```

6 . For the rest, you just have to do this
```bash
pip install MDAnalysis
```

## How-To

1 . First you need to extract all the chemical shift from MD trajectory (or PDB files). You need at least to specify a PDB directory or a topology and a dcd file (**don't forget to rename all the special residues like HS[EDP] to HIS in the topology file, because SHIFTX+ won't recognize them**). Prediction from PDB files will use only one processor, so you don't have to use MPI. On the contrary, to predict all the chemical shift from MD trajectories I recommand you to use MPI (knowing that SHIFTX+ is not the fastest man alive), especially if they are very long (> 100000 frames).
```bash
mpiexec -np 4 python shift.py -t topology.psf -d traj.dcd
```
**Command line options**
* -p/--pdb: directory with pdb files
* -t/--top: topology file (pdb, psf)
* -d/--dcd: single trajectory of list of trajectories (dcd, xtc)
* --pH: pH (default: 7.4)
* --temperature: temperature (default: 7.4)
* -i/--interval: used frames at this interval (default: 1)
* -s/--shift: shift the resid number to match with Xray or experimental data (default: 0)

2 . And finally, compare them to experimental data. It will compute the RMSD between the prediction and the experimental data and plot the secondary chemical shift along the sequence for each element (Ca, Cb, etc ...). Finally, if you want, you can plot the chemical shift distribution for each residue (long operation).
```bash
python analyze.py -c obs.bmrb -h5 shiftx.hdf5 -d dssp.file
```
**Command line options**
* -c/--obs: BMRB file with experimental chemical shift
* -h5: HDF5 file with the chemical shift
* -d/-dssp: DSSP file to plot secondary structure on the plot (Default: None)
* --distribution: if you wan to plot the chemical shift distribution for each residue (default: False)

## Citation
1. Beomsoo Han, Yifeng Liu, Simon Ginzinger, and David Wishart. (2011) SHIFTX2: significantly improved protein chemical shift prediction. Journal of Biomolecular NMR, Volume 50, Number 1, 43-57. doi: 10.1007/s10858-011-9478-4.


## License
MIT