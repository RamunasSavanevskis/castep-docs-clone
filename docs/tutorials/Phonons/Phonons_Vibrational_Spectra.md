# Phonons and Spectroscopy

In this tutorial we will perform Castep phonon calculations. We will visually examine the vibrations and then examine, and compare to experiment, the IR and Raman spectra.

## G-point Phonon Spectrum of h-BN

We will first perform a G point phonon calculation and generate a simple model infrared spectrum of hexagonal boron nitride (h-BN). We will then visualise and analyse the results.

We will use the `cell` file

*hbn.cell*
```
%BLOCK LATTICE_ABC
2.500375 2.500375 6.635274
 90.0000000  90.0000000 120.0000000
%ENDBLOCK LATTICE_ABC

%BLOCK POSITIONS_FRAC
B       0.0000000   0.0000000   0.5000000
B       0.3333000   0.6667000   0.0000000
N       0.0000000   0.0000000   0.0000000
N       0.3333000   0.6667000   0.5000000
%ENDBLOCK POSITIONS_FRAC

SYMMETRY_GENERATE

%BLOCK SPECIES_POT
NCP
%ENDBLOCK SPECIES_POT

KPOINTS_MP_GRID 7 7 4
PHONON_KPOINT_MP_GRID 1 1 1
```

and the `param` file

*hbn.param*
```
task : PHONON
phonon_method : DFPT
xc_functional : LDA
opt_strategy : SPEED
cut_off_energy : 700.0 eV
fix_occupancy : TRUE
elec_method : DM
phonon_sum_rule : TRUE
```

A key point to make is that, as specified in the `cell` file, we are using the norm-conserving pseudopotentials (`NCP`) library - this is essential to allow the use the `DFPT` method, which allows calculations outside the cell without using a supercell - something that must be done for proper phonon calculations in a crystalline system.    

The line

`phonon_kpoint_mp_grid 1 1 1`

is used to specify the gamma-point phonon wavevector.

Once the files are set up, run Castep

!!! note
    Phonon calculations can take a while, so it is recommended to use multiple cores if possible

This generates 2 files in which we are interested: the standard `hbn.castep` output and `hbn.phonon` for phonons specifically. Let's first look at `hbn.castep`.

All the data relevant to the phonon calculations is found under the header

```
==============================================================================
+                           Vibrational Frequencies                          +
+                           -----------------------                          +
```

and, since we are only looking at the gamma point, all our results are under

```
+ -------------------------------------------------------------------------- +
+  q-pt=    1 (  0.000000  0.000000  0.000000)     1.0000000000              +
+ ---------------------------------------------------------------------------
```

The main table of results relevant to this tutorial is this

```
+  Acoustic sum rule correction <  26.548113 cm-1 applied                    +
+     N      Frequency irrep.    ir intensity active            raman active +
+                (cm-1)         ((D/A)**2/amu)                               +
+                                                                            +
+     1     -26.582880   a          0.0000000  N                       Y     +
+     2     -26.582880   a          0.0000000  N                       Y     +
+     3      -0.049168   b          0.0000000  N                       N     +
+     4      -0.034768   c          0.0000000  N                       N     +
+     5      -0.034768   c          0.0000000  N                       N     +
+     6      77.735088   d          0.0000000  N                       N     +
+     7     748.297431   b          4.3146565  Y                       N     +
+     8     802.240346   d          0.0000000  N                       N     +
+     9    1392.383294   a          0.0000000  N                       Y     +
+    10    1392.383294   a          0.0000000  N                       Y     +
+    11    1392.756214   c         54.8065334  Y                       N     +
+    12    1392.756214   c         54.8065334  Y                       N     +
```

The frequencies are the frequencies of all the phonon modes. Note modes 3, 4 and 5 - these are all close to 0, indicating that they are the acoustic modes (we will understand what this means better once we visualise it in the next step). The "negative" frequencies aren't actually negative but rather imaginary - this means that, when calculated, the vibrational hamiltonian is not a definite positive value, and thus the distortion would stabilise the structure - this corresponds to a mechanical instability in the system.

Each phonon frequency is then evaluated as to whether it is IR and/or Raman active, and, for IR only for now, the relative intensity of the IR activity. 

### Analysis of h-BN phonon output.

We will use the academic tools to visualise the modes using the free
Jmol visualiser.

Use secure file transfer to copy the .phonon file back to the PC where
Jmol is installed. Start jmol and bring up a console window from the
right-mouse menu. To use the full power of the command-line and find the
files it is most convenient to set JMol's current directory to the
location of your working directory containing your output files. (It is
possible and simpler to drag and drop the .phonon file onto Jmol window,
but you need the power of the command line to display additional
periodic repeats.)

> cd

> --- report the current directory

> *cd ..* or c*d \<subdir\>* -- navigate up/down the directory tree.

Then type the Jmol command to load your .castep file

> <span id="anchor"></span>*load h-BN.phonon {3 3 2} PACKED*

Note that while you can read in the .phonon file from Jmol's File menu,
you need the additional opitions of the command line to display
additional periodic repeats.

You can then use the “Tools-Vibrate” menu to turn on mode animation, and
navigate the modes. Can you see from the mode eigenvectors which modes
are ir active and which are raman active? Do you agree with the ir and
raman activity printed in the .castep file?

### Generation of ir spectrum

The easiest way to generate a simple model ir spectrum is to use the
“dos.pl” tool. To run this in the most effective way and automatically
display a plot, you will need to be running an X windows server on the
PC. In that case the command

> *dos.pl -ir -xg h-BN.phonon*

will generate a plot script and use the “xmgrace” plotting program to
display it. You can create a GNUPLOT script instead of xmgrace by
changing the “-xg” flag to “-gp”. Alternatively you can generate a
GNUPLOT script without plotting by

> *dos.pl -ir -gp -np h-BN.phonon \> h-BN-phonon.plt*

You can then copy the “h-BN-phonon.plt” back to the PC and read this
into GNUPLOT.

### Raman spectrum

The calculation of a raman intensities is fairly expensive compared to
infrared matrix elements and it is therefore not turned on by default.
To enable this, add the keyword

 * CALCULATE_RAMAN : TRUE*

RAMAN_METHOD: DFPT

FIX_OCCUPANCY: TRUE

to the h-BN.param file, and resubmit the job. You can then use the
“-raman” flag of dos.pl to generate and plot a raman spectrum.

## B. Molecular modes in benzene

The next part of this practical is to compute the modes and spectrum of
a molecule and compare the result with a calculation of a molecular
crystal. Our example is benzene. You are supplied with pdb files
describing a benzene molecule and of a cif structure describing a
high-pressure crystalline polymorph, phase III. These are in both in the
course directory.

First run the isolated molecule calculation. Using the supplied PDB
format file and using the “pdb2cell” utility, generate a .cell file for
a single molecule calculation. There are a few other considerations to
take into account for an isolated molecule calculation.

1.  The size of the simulation cell governs the interactions between
    periodic copies of the molecule and should be large enough that
    these are negligible.
2.  The shape of the simulation cell governs the crystallographic point
    groups allowed in the handling of the symmetry. It should be chosen
    to be commensurate (as far as possible) with the molecular point
    group to maximise the use of symmetry. In the case of benzene it
    should obviously be hexagonal, and I recommend a box 8A by 8A by 4A.
3.  Recall that there is no electronic dispersion for a molecule, so
    only a single electronic k-point is needed. The general rule is that
    all “molecule in a box” calculations should use the G point only as
    CASTEP uses special performance optimisations in this case.

For simplicity use the local density approximation, the standard NCP
library pseudopotentials.

In benzene not all atoms are on special symmetry sites so you should
first perform a geometry optimisation. Then set up a follow-on
calculation to compute the Gamma point phonons. The Phonons User Guide
on the WWW should help you fill in the details, but please ask if you
are stuck.

Benzene Phase III: The next stage is to compute the gamma point phonon
modes of the molecular crystal of benzene in the high pressure
polymorph, Phase III. You are supplied with a .cif file. You should use
a 2x2x2 grid of electronic k-points, as dispersion is non-zero in this
molecular crystal. Make sure that symmetry is detected and enabled. Use
the same .param file as for the molecular case to ensure the settings
are the same.

Once these calculations have completed you should generate a phonon DOS
and ir spectra as in the previous practical and compare the molecule
with the molecular crystal. You can also use Jmol to identify the modes.

Are all your frequencies positive? If not, can you suggest why not? Try
investigating the effect of decreasing the geometry optimisation
tolerance GEOM_FORCE_TOL. How does this change the frequencies?
