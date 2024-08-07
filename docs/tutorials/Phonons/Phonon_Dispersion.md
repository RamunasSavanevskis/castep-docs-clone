# C. Phonon dispersion using interpolation in NaH

From G point only calculations we now explore the whole of the Brillouin
zone of phonon wavevectors. Our example is the rocksalt-structured
hydride, NaH which should run quickly enough to return results in a few
minutes. Based on previous exercises, lectures and the user manual, you
should be able to set up and run a DFPT phonon dispersion calculation
and display a well converged set of dispersion curves.

Some suitable settings are

- The primitive fcc unit cell of the B1 rocksalt structure has
  a=b=c=3.393405, ===60, with the Na ion at (0,0,0) and the H ion at
  (1/2,1/2,1/2). This is a high-symmetry structure so it is important to
  instruct CASTEP to generate and use the full symmetry set.

- Use norm-conserving pseudopotentials

  %block species_pot

NCP

%endblock species_pot

in the .cell file and “basis_precision : fine” in the .param file.

- A suitable phonon qpoint grid for the interpolation is an offset 4x4x4
  grid

   * phonon_kpoint_mp_grid 4 4 4*

  phonon_kpoint_mp_offset 0.125 0.125 0.125

- A suitable list of points for the fine q-point path for FCC is

   * %block phonon_fine_kpoint_path*

  0.0 0.0 0.0 ! Gamma

  0.5 0.5 0.0 ! X (along Delta

  1.0 1.0 1.0 ! Gamma (Sigma )

  0.5 0.5 0.5 ! L (Delta)

  0.5 0.75 0.25 ! W (Q)

  0.5 0.5 0.0 ! X (Z)

  %endblock phonon_fine_kpoint_path

You will first need to perform a k-point convergence test. For a phonon
calculation, convergence of the *forces* is an appropriate test
criterion. Since all of the ions are on symmetry positions the forces
are zero by symmetry. Try to think of a way around this obstacle. And
adopt a suitable compromise between accuracy and run-time.

## C.II Phonon DOS using interpolation

The task here is to use the NaH example to compute and display not a
dispersion curve but a density of states. This will exploit CASTEPs
interpolation functionality, and you will be able to compute a good DOS
without the need the repeat the expensive electronic structure
calculation. To do this you will need to set up a calculation neatly
identical to your previous one but with two differences.

1.  The calculation should be set up as a continuation. If your previous
    run wrote a .check file named “NaH-disp.check” for example, then the
    param file should contain the line

     * continuation : NaH-disp.check*

2.  Instead of a %block phonon_fine_kpoint_list in the .cell file, you
    can specify a grid

    *phonon_fine_kpoint_mp_grid 16 16 16*

You can run CASTEP on just 1-4 processors for this

You can try this several times with different fine q-point grids.

This will produce a .castep and .phonon file as before. You may analyse
the .phonon file and generate a DOS using the “dos.pl” script

> *dos.pl -xg NaH-dos.phonon*

(again, an X server running on the PC will be needed for grace to
display) .
