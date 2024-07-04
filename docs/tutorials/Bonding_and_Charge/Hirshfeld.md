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
