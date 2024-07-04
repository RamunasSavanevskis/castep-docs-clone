Whereas in the [Mulliken population analysis tutorial](mulliken_population.md) we performed population analysis using the [Mulliken algorithm](../../documentation/Groundstate/population_analysis.md#mulliken-population-analysis) (which is done by default), we will now use the [Hirshfeld algorithm](../../documentation/Groundstate/population_analysis.md#the-hirshfeld-partitioning-scheme) to get more and different results.

!!! note

    It is recommended you go through the [previous (Mulliken) tutorial](mulliken_population.md) as the steps are very similar


## Si and GaAs

We will compare Si and GaAs

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
For `GaAs.cell` you may use
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
<a name="param"></a>
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

The numbers alone don't tell us much, but we can make some interesting conclusions if we compare it to GaAs

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

In GaAs, the 2 ions are now different and must be looked at separately - Ga has a contraction of 5.77(ANGSTROMS) - 4.9% while As contracts by 3.9(ANGSTROMS) - 3.5%. In addition, the Ga and As ions now have charges (again intuitively and also found in Mulliken).

For both ions, the contraction is smaller than Si's, indicating a weaker bond - which was shown [before](mulliken_population.md) as well. However, the Ga's atomic volume decreases more than the As's. This is proof that there is a level of ionic bonding/polarisation as well. While they both contract due to the covalent bond, electrons are more likely to be found near the As (which is more electronegative) than the Ga, meaning that the As's atomic volume is higher than it would be if the electron distribution was equal for both ions.

<a name="GaAs_charges"></a>
Another thing to note is that the Hirshfeld charges are different than the Mulliken ones- for example, for GaAs they are 0.11e in Hirshfeld calculation but 0.083e in the [Mulliken](mulliken_population.md) one. COMMENT ON LATER??

## Diatomic molecules

We will examine the same diatomic molecules as before - HF, HCl and HBr - and see if the results agree with what the calculations said there.

The `cell` files we use will be the same (ADD LINK - think its a waste to write them out again) and we will use the same `param` file for all of them as [above](#param)

The results are as follows:

*HF.castep*

```
Species     1,  Atom     1  :  H
...
 Free atom volume (Bohr**3) :
                                      11.282018688
 Hirshfeld total electronic charge (e) :
                                       0.000000000
 Hirshfeld net atomic charge (e) :
                                       0.209852855
 Hirshfeld atomic volume (Bohr**3) :
                                       6.747865069
 Hirshfeld / free atomic volume :
                                       0.598107950

Species     2,  Atom     1  :  F
...
 Free atom volume (Bohr**3) :
                                      20.173660282
 Hirshfeld total electronic charge (e) :
                                       0.000000000
 Hirshfeld net atomic charge (e) :
                                      -0.211251954
 Hirshfeld atomic volume (Bohr**3) :
                                      19.218247638
 Hirshfeld / free atomic volume :
                                       0.952640590


    Hirshfeld Analysis
    ------------------
Species   Ion     Hirshfeld Charge (e)
======================================
 H        1                 0.21
 F        1                -0.21
======================================
```

*HCl.castep*

```
Species     1,  Atom     1  :  H
...
 Free atom volume (Bohr**3) :
                                      11.282018688
 Hirshfeld total electronic charge (e) :
                                       0.000000000
 Hirshfeld net atomic charge (e) :
                                       0.107816784
 Hirshfeld atomic volume (Bohr**3) :
                                       8.640441867
 Hirshfeld / free atomic volume :
                                       0.765859560

Species     2,  Atom     1  :  Cl
...
 Free atom volume (Bohr**3) :
                                      65.515626441
 Hirshfeld total electronic charge (e) :
                                       0.000000000
 Hirshfeld net atomic charge (e) :
                                      -0.107842259
 Hirshfeld atomic volume (Bohr**3) :
                                      63.675188092
 Hirshfeld / free atomic volume :
                                       0.971908406


    Hirshfeld Analysis
    ------------------
Species   Ion     Hirshfeld Charge (e)
======================================
 H        1                 0.11
 Cl       1                -0.11
======================================
```
*HBr.castep*

```
Species     1,  Atom     1  :  H
...
 Free atom volume (Bohr**3) :
                                      11.282018688
 Hirshfeld total electronic charge (e) :
                                       0.000000000
 Hirshfeld net atomic charge (e) :
                                       0.080189170
 Hirshfeld atomic volume (Bohr**3) :
                                       9.207861059
 Hirshfeld / free atomic volume :
                                       0.816153679

Species     2,  Atom     1  :  Br
...
 Free atom volume (Bohr**3) :
                                      89.482329717
 Hirshfeld total electronic charge (e) :
                                       0.000000000
 Hirshfeld net atomic charge (e) :
                                      -0.080194171
 Hirshfeld atomic volume (Bohr**3) :
                                      87.876549029
 Hirshfeld / free atomic volume :
                                       0.982054773


    Hirshfeld Analysis
    ------------------
Species   Ion     Hirshfeld Charge (e)
======================================
 H        1                 0.08
 Br       1                -0.08
======================================
```

Comparing to the [Mulliken analysis](mulliken_population.md) (the results of which are again present in the same `castep` file), you may notice that the Hirshfeld charges are smaller than the Mulliken ones (as opposed to [GaAs](#GaAs_charges), where they were larger) - ((EXPLAIN WHY??))

Next let's look at the relative contractions for all of them.

* HF: H -40.2%, F -4.7%
* HCl: H -23.4%, Cl -2.8%
* HBr: H -18.4%, Br -1.8%

Like before, this is indicative of covalent bonding: since they both contract (including the electronegative species) that means that electron shell is moving from the individual atoms into the bond. However, like in GaAs, there is a difference between the electropositive and electronegative species due to the electrons being more likely to be distributed around the electronegative one, effectively growing it while shrinking the electropositive one.

In this, there is again an indication of decreasing electronegativity as you go down the periodic table: the contraction of hydrogen decreases, which is evidence of hydrogen's electron being less likely to be at the halogen - meaning its electronegativity is lower

### Issues

The main problem with the above analysis is that Hirshfeld does not say where the atoms go - the hydrogen may be shrinking more because the electronegativity difference is higher and its electron is less likely to be near it, or it could also be because the covalent bond is getting weaker so it's less likely to be found in the bonding region.

The halogen shrinkage is hard to interpret as well. A smaller % contraction can either mean it's more electronegative (contradicting what's known and the above analysis), weaker covalent bonding, and/or, as is the case here, simply because the atom is larger - the more inner shells there are, the less affected the atom volume will be by what the outer shell electrons are doing.
