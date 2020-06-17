# PiSCF - Quantum Chemistry on a Raspberry Pi

What? Running PySCF on a Pi (geddit)
Why? Quantum chemistry calculations are fun and I don't understand them as much as I should do.
Gaussian and Spartan make it easy to perform calculations, while somewhat removed from the
underlying theory. Also, it seemed like a good excuse to brush up on Python and Linux command 
line skills.

## Installing PySCF

Initial attempts to install PySCF on a Pi3B was unsucccessful. Getting the PySCF installation 
to find libcint seemed to be beyond my minimal Linux capabilities, and when I tried to build 
PySCF from Github the process failed due to insufficient memory.

To address the memory problem...well, along came my birthday and with it, a 4GB Pi4!

The following steps detail the successful PySCF installation on a fresh install of Raspberry
Pi OS using a Pi4 (4GB) and a 32GB SD card (just what I had lying around).

```
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install git cmake python-h5py python-scipy
```

I don't think you need both of these BLAS libraries and there's a way to switch between
them but again, limited knowledge currently fails me. Currently looking into this...

```
$ sudo apt install libatlas-base-dev
$ sudo apt install libatlas-doc
$ sudo apt install libopenblas-base
$ sudo apt install libopenblas-dev
```

```
$ git clone https://github.com/sunqm/pyscf
$ cd pyscf
$ cd pyscf/lib
$ mkdir build
$ cd build
$ cmake ..
$ make
$ echo 'export PYTHONPATH=~/pyscf:$PYTHONPATH' >> ~/.bashrc
$ source ~/.bashrc
```


## Benchmarking

I am under no illusion that a Raspberry Pi is going to compete with a Xeon processor, even
one from many generations ago, yet I have to know! I created a short Python snippet to run
tests on a range of different Pis - it is a very simple example of a DFT calculation of
bezene.

```python
from datetime import datetime
from pyscf import gto,dft

mol = gto.M(atom = '''\
 C 3.36990590    0.32915360    0.00000000\
 C 4.76506590    0.32915360    0.00000000\
 C 5.46260390    1.53690460    0.00000000\
 C 4.76494990    2.74541360   -0.00119900\
 C 3.37012490    2.74533560   -0.00167800\
 C 2.67252390    1.53712960   -0.00068200\
 H 2.82014690   -0.62316340    0.00045000\
 H 5.31457390   -0.62335940    0.00131500\
 H 6.56228390    1.53698460    0.00063400\
 H 5.31514990    3.69755660   -0.00125800\
 H 2.82000290    3.69761660   -0.00263100\
 H 1.57291990    1.53731260   -0.00086200\
''', basis = 'cc-pvdz')

mf = dft.RKS(mol)
mf.xc = 'HYB_GGA_XC_B3LYP'

ti = datetime.now()
energy = mf.kernel()
tf = datetime.now()

print("Benzene energy = {:.9g} hartrees".format(energy))
print("Calculation time: {}".format(tf-ti))
```


### Raspberry Pi 4 (4GB)

No active cooling used, but the Pi has one of those large heatsink cases.

1. The very first test actually used the cc-pvtz basis set although this very quickly 
resulted in a ValueError indicating a NumPy array was too big (memory issue?).

2. Using the cc-pvdz basis set (for this and all subsequent tests, the calculation took 
44 seconds. The maximum temperature reached was 42 degrees celcius, all cores were utilised, 
and maximum memory usage was observed around 20% (using `top`).

Test after installation of PySCF on 64 bit OS

1. 50 seconds

2. Using cc-pvtz, 6 minutes 16 seconds... it didn't throw an error!


### Raspberry Pi 3 Model B (v1.2)

Again, no active cooling used, but the Pi had a set of small heatsinks stuck on.

1. Run took 1 minute 18 seconds. Max. temperature was 53 degrees. All cores utilised and
maximum memory usage often hit 81%.
2. Started only moments later, the seocnd run took 1 minute 27 seconds. Max. temperature
was 57 degrees.
3. Third run in quick succession did not complete. NumPy ValueError - some array contained
infs or NaNs.
4. The Pi Crashed. It was never the same again.

I didn't bother testing any of my other Pis (zero, Pi1B, P1A) and am also yet to try the 64 bit OS with the Pi3B.

## The Dream

A cluster of five Raspberry Pi's, four of which are Pi4 (8GB obviously) compute nodes.
Each one has that miniature tower cooler and they're all submerged in mineral oil.

No, it would't be the most capable machine for the money but it would be *damn cool*.
