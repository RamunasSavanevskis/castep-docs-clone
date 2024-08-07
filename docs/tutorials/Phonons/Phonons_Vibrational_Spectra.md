# Phonons and Spectroscopy

In this tutorial we will perform Castep phonon calculations. We will visually examine the vibrations and then examine, and compare to experiment, the IR and Raman spectra.

## G-point Phonon Spectrum of h-BN

We will first perform a G point phonon calculation and generate a simple model infrared spectrum of hexagonal boron nitride (h-BN). We will then visualise and analyse the results.

Your starting point will be the structure which is provided in a cif
file named *h-BN.cif.* The Phonons manual
(<http://www.tcm.phy.cam.ac.uk/castep/Phonons_Guide/Castep_Phonons.html>)
contains a very similar example for the Wurtzite-structure polymorph of
BN.

Use the *cif2cell* program to generate a .cell file:run the command

> cif2cell -p castep h-BN.cif \> h-BN.cell

then edit this using one of the text editors installed on the system.
Uncomment the “%block species_pot” and change it to read

- %block species_pot

NCP

%endblock species_pot

to select the Norm-conserving pseudopotentials library .

Choose the k-point set to use – remove any block KPOINT_LIST and replace
with the line

* kpoint_mp_grid 7 7 4*

and one to specify a gamma-point *phonon* wavevector

* phonon_kpoint_mp_grid 1 1 1*

Test your configuration using

* castep.mpi –dryrun h-BN*

Congratulations. You have set up the .cell file by hand. There is also a
*.param* file “h-BN.param” provided containing the setup to specify the
phonon run.

Once you have in place the \<seed\>.cell and \<seed\>.param files you
are ready to submit the CASTEP job. This is done using our general
script:

> *castepsub -n 16 h-BN*

which requests a 16-processor parallel run. Use the “qstat” command to
monitor the progress of your calculation. When it has finished, you can
examine the output file h-BN.castep and find the frequencies. What you
see is explained further in the Phonons user guide on the WWW. There is
also a machine-readable file h-BN.phonon which contains the frequencies,
but also the eigenvectors which we will analyse.

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
