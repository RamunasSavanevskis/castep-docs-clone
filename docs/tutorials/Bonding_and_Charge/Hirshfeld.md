Whereas in the [Mulliken population analysis tutorial](mulliken_population.md) we performed population analysis using the [Mulliken algorithm](../../documentation/Groundstate/population_analysis.md#mulliken-population-analysis) (which is done by default), we will now use the [Hirshfeld algorithm](../../documentation/Groundstate/population_analysis.md#the-hirshfeld-partitioning-scheme) to get more and different results.

!!! note

    It is recommended you go through the [previous (Mulliken) tutorial](mulliken_population.md) as the steps are very similar


## Diamond structures

We will compare 4 different diamond structures - Si, diamond (carbon-diamond) and GaAs - the same as before.

For Si, we will use the same `cell` file

`silicon.cell`

```
%block lattice_abc
3.8 3.8 3.8
60 60 60
%endblock lattice_abc
!
! Atomic co-ordinates for each species.
! These are in fractional co-ordinates wrt to the cell.
!
%block positions_frac
Si 0.00 0.00 0.00
Si 0.25 0.25 0.25
%endblock positions_frac
!
! Analyse structure to determine symmetry
!
symmetry_generate
!
! Specify M-P grid dimensions for electron wavevectors (K-points)
!
kpoint_mp_grid 4 4 4

```
Again, the `diamond.cell` file will be identical except the length in the `lattice_abc` block will be 2.52(ANGSTROM INSERT LATER) rather than 3.8. For `GaAs.cell` you may use
```
%BLOCK LATTICE_CART
2.825 2.825 0.0
2.825 0.0 2.825
0.000 2.825 2.825
%ENDBLOCK LATTICE_CART
%BLOCK POSITIONS_FRAC
Ga 0.0 0.0 0.0
As 0.25 0.25 0.25
%ENDBLOCK POSITIONS_FRAC
```  
The `param` files we will use are all identical -
```
xc_functional : LDA
cutoff_energy : 500 eV
spin_polarised : false
CALCULATE_HIRSHFELD : TRUE
IPRINT : 2
```
There are 2 distinct differences to [before](mulliken_population.md). For one, the line

```
CALCULATE_HIRSHFELD: TRUE
```

ensure that it also performs a Hirshfeld calculation (it will still perform Mulliken as well). The line

```
IPRINT : 2
```

Will give more information - without it only the Hirshfeld charges will be added, while this gives more interesting information such as atomic volumes.

After running castep for the structures, we see towards the end of the `.castep` output files:

*Si.castep*

```
Species     1,  Atom     1  :  Si
...
Free atom volume (Bohr**3) :
                                    105.995197080
Hirshfeld total electronic charge (e) :
                                      0.000000000
Hirshfeld net atomic charge (e) :
                                      0.000001738
Hirshfeld atomic volume (Bohr**3) :
                                     99.207952643
Hirshfeld / free atomic volume :
                                      0.935966491
...
Species     1,  Atom     2  :  Si
...
Free atom volume (Bohr**3) :
                                    105.996110529
Hirshfeld total electronic charge (e) :
                                      0.000000000
Hirshfeld net atomic charge (e) :
                                     -0.000001738
Hirshfeld atomic volume (Bohr**3) :
                                     99.209046098
Hirshfeld / free atomic volume :
                                      0.935968741


   Hirshfeld Analysis
   ------------------
Species   Ion     Hirshfeld Charge (e)
======================================
Si       1                 0.00
Si       2                -0.00
======================================

```

As expected, both atoms are identical and the Hirshfeld charges are 0 - this is already known intuitively and from the Mulliken analysis.

However, this contains some new data - Free atom volume and Hirshfeld atomic volume. The atomic volume is about 6.79(ANGSTROMS) smaller, corresponding to a contraction of 6.4%.

The numbers alone don't tell us much, but we can make some interesting conclusions if we compare it to the other diamond structures:

*diamond.castep*
```
Species     1,  Atom     1  :  C
 ...
 Free atom volume (Bohr**3) :
                                      38.933903844
 Hirshfeld total electronic charge (e) :
                                       0.000000000
 Hirshfeld net atomic charge (e) :
                                      -0.000004409
 Hirshfeld atomic volume (Bohr**3) :
                                      35.367220474
 Hirshfeld / free atomic volume :
                                       0.908391324
```

*GaAs.castep*

```
Species     1,  Atom     1  :  Ga
...
 Free atom volume (Bohr**3) :
                                     117.353896691
 Hirshfeld total electronic charge (e) :
                                       0.000000000
 Hirshfeld net atomic charge (e) :
                                       0.111060719
 Hirshfeld atomic volume (Bohr**3) :
                                     111.582933619
 Hirshfeld / free atomic volume :
                                       0.950824274

Species     2,  Atom     1  :  As
...
 Free atom volume (Bohr**3) :
                                     110.726221123
 Hirshfeld total electronic charge (e) :
                                       0.000000000
 Hirshfeld net atomic charge (e) :
                                      -0.111060719
 Hirshfeld atomic volume (Bohr**3) :
                                     106.824797235
 Hirshfeld / free atomic volume :
                                       0.964765131


    Hirshfeld Analysis
    ------------------
Species   Ion     Hirshfeld Charge (e)
======================================
 Ga       1                 0.11
 As       1                -0.11
======================================
```

The carbon atoms in diamond have a contraction of about 3.57(ANGSTROMS), corresponding to about 9.2%. In GaAs, the 2 ions are now different and must be looked at separately - Ga has a contraction of 5.77(ANGSTROMS) - 4.9% while As contracts by 3.9(ANGSTROMS) - 3.5%. In addition, the Ga and As ions now have charges (again intuitively and also found in Mulliken).
