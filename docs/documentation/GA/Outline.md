# castep~GA~ Outline

## Introduction


## GA Overview

The general process is outlined below:

1.  The GA is initialised by calling it with a seed for which cell and
    param files exist. I.e. `castep_GA seed`

2.  The `<seed>.cell` and `<seed>.param` files are read into the GA. An
    initial population of size `ga_pop_size` is created by performing
    mutation with large amplitudes and probabilities on multiple copies
    of the cell given in `<seed>.cell`. This creates the first
    (generation 0) parent population.

3.  The initial parent population is relaxed using a standard CASTEP
    geometry optimisation using the parameters in `<seed>.param`.

    -   If the symmetry snap method is enabled then after the initial
        cell relaxation the following steps are carried out:
        1.  The relaxed cell is snapped to symmetry sites within
            `symmetry_tol`.
        2.  The cell snapped to symmetry sites is relaxed again using
            the parameters in `<seed>.param`.

4.  Final relaxed cells from the normal CASTEP geom-opt are read back
    into the GA.

5.  Each structure is now evaluated for its fitness. This is a function
    of the final enthalpy. Cells that fail to converge are assigned a
    high enthalpy.

6.  Once all the population members in the current generation have been
    relaxed, roulette selection based upon fitness is used to choose
    which pairs of cells to become parents.

7.  Two parents then breed to generate two children. There is also some
    random mutation in the children at this stage. This repeats until
    `ga_pop_size` children have been created.

8.  The child population is relaxed using a standard CASTEP geometry
    optimisation using the parameters in `<seed>.param` as in steps 3-5
    above.

9.  The union of parent and child populations is now trimmed using the
    fitness function and roulette selection until `ga_pop_size` members
    remain.

10. Steps 6-9 are repeated until global convergence is achieved, defined
    by a number of generations with no change in members of
    non-dominated front.

## Serial and Parallel Calculations

The `castep_GA` is a serial program, and must be ran in serial in order
to allow for seeding further CASTEP calls. These CASTEP calls can either
be synchronous (i.e. the first must finish before the next one can
start) or asynchronous (multiple CASTEP jobs running at the same). In
addition, each CASTEP job itself may be either serial or parallel (with
the usual range of parallelization schemes available). This can be
controlled by the user, depending on the size of the computing facility
and the number of population members required.

The command used to call CASTEP for the cell relaxations is given in a
devel code block in between the flags `CMD:` and `:ENDCMD`, though
default is `castep.serial` parralelised methods such as
`mpirun castep.mpi` can also be used. See
[Command for calling castep](Input_Options.md#command-for-calling-castep-cmd)
