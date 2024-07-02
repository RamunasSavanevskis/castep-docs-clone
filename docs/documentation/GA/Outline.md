# castep~GA~ Outline

## Introduction

The standard geometry optimization routines in CASTEP only perform a
local optimiziation. That is, the final state might not be the global
minimum enthalpy structure. Hence, the final structure found will be the
optimal state that exists within the basin of attraction of the initial
state. If working for example from an initial guess at a structure
guided by experimental knowledge or a known structure from ICSD etc,
this is often sufficient. However, if there is no experimental data to
guide, then this is just guesswork. Hence there is a real need for a
global optimization approach, that can reliably find the global optimal
structure without a priori information. There are various approaches to
this problem, including Chris Pickard\'s AIRSS (Ab Initio Random
Structure Search) and the Genetic Algorithm (GA) approach used here.
Both of these different approaches use CASTEP for the local optimization
but differ in their approach:

-   AIRSS uses many different random guesses at an initial structure
-   GA generates an initial population of guesses and then uses
    evolutionary ideas to breed and mutate these so that successive
    generations evolve into better structures.

The first application of the GA idea to crystal structure prediction was
[^1] which was then extended to include a \'fingerprint\' method to
enhance structural diversity and prevent stagnation [^2]. A full
explanation of this initial version can be found in the PhD thesis of
Luke Abraham [^3]. This initial version (v1) of castep~GA~ was not
released, as it was not very robust. The code was tightly integrated
with the rest of CASTEP, so that if any one of the structures
(population member) within the different generations failed to converge,
the whole calculation aborted. This led to a signicant codebase change
with v2 now \"spawning\" seperate CASTEP instances to do the local
geometry optimization. The code was still built upon the CASTEP
codebase, but was now robust to any structures failing to converge. This
version also contained new features to handle spin, so that castep~GA~
could now breed (vector) spin structures in addition to atomic
structures using a new \'spin fingerprint\' algorithm [^4]. This is
fully explained in the PhD thesis of Ed Higgins [^5]. As well as being
able to search for bulk crystal structures, the castep~GA~ can also
search for interface or surface reconstructions - an example being the
Si\[111\](7x7) reconstruction studied in [^6]. Finally, in v3, the
ability to add random cells and cells with randomly chosen space groups
were added and the Multi Objective GA (MOGA) approach was implemented,
as described in the seperate readme~MOGA~ docs by Scott Donaldson. This
enables the user to search for structures that are Pareto optimal with
an abitrary choice of optimization functions via a flexible script-based
approach. Further details can be found in \[fn scott report\]. This is
the basis of the first version of castep~GA~ for general release.

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
    optimisation using the parameters in `<seed>.param` as in steps 3-5i
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
[*Command for Calling Castep (\~CMD\~)*]{.spurious-link
target="Command for Calling Castep (~CMD~)"}.
