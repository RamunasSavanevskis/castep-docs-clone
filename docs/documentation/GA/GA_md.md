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

# Requirements

A working version of CASTEP is required, that will be called with the
command given in-between the `CMD:` and `:ENDCMD` flags in the devel
code block. For example with `CMD: castep.serial :ENDCMD` or
`CMD: mpirun castep.pmi :ENDCMD`.

The command line tools sed, grep and awk must be installed.

## Param File Requirements

The GA must be enabled using the `.param` file option
`task = genetic algor` and the `GA=T` flag set in the `GA: :ENDGA` devel
code block.

As the GA will edit the base `.param` file as part of the spawning
CASTEP jobs, the `.param` file is required to end with one empty line.

## Compiling `castep_GA`

Compiling can now be automatically carried out simply by calling \'make
tools\' from the CASTEP top-level directory.

## License

The GA is built using many standard CASTEP source code routines, and so
is distributed as part of the CASTEP source. Consequently, it is covered
by the same CASTEP licence terms and is NOT for general distribution.

# Input Options

## Calling the GA

After compiling the GA with `make tools` the GA program will be placed
within the relevant directory for your system within the `obj` directory
(which is within the CASTEP directory from where `make tools` is ran).

To use the GA this program (not CASTEP itself) must be executed. If this
directory is placed in the `$PATH` then use `castep_GA <seed>` where
`<seed>` is the relevant seed. For example for input files with the seed
`Si` the following command should be used:

    castep_GA Si

Note that `castep_GA` is a serial program that will seed CASTEP
calculations based on the command given in the `CMD` devel flags (see
[*Serial and Parallel Calculations*]{.spurious-link
target="Serial and Parallel Calculations"},
[*Requirements*]{.spurious-link target="Requirements"} and
[*Command for Calling Castep (\~CMD\~)*]{.spurious-link
target="Command for Calling Castep (~CMD~)"}), if not given all seeded
calculations will be serial called with `castep.serial`.

## General Inputs - Normal Parameters - GA

General inputs for the GA that may be set in the `<seed>.param` file:

`task = genetic algor`
:   turn on the GA code. This is changed within the castep~GA~ code to
    `task = geometry optimization` to do the separate geom-opts.

`ga_pop_size (integer)`
:   Number of members in a population at the start of each generation.

`ga_max_gens (integer)`
:   Maximum number of generations to run for, if convergence is not
    reached.

`ga_mutate_rate (real)`
:   Probability of mutating a given atomic position or cell vector.

`ga_mutate_amp (real)`
:   Maximum size of an atomic position or cell vector mutation, in
    Angstroms.

`ga_spin_mutate_rate (real)`
:   Probability of mutating a given atomic spin.

`ga_spin_mutate_amp (real)`
:   Maximum size of the resultant atomic spin after mutation.

`ga_fixed_n (logical)`
:   Whether or not the number of atoms per member can be varied (true
    -\> atoms/member is fixed).

`ga_bulk_slice (logical)`
:   Whether or not to perform crossover as if the material is bulk-like
    and hence as 2 parallel cuts (true) or lower-dimensional and hence
    has only 1 cut (false).

## General Inputs - Devel Block - core GA

Additional functionality is available within the \"devel block\" part of
the `<seed>.param` file. This is beta functionality, that may be removed
in the future, or promoted to full CASTEP keyword status:

`PP (logical)`
:   Use pair-potentials e.g. for testing or full DFT (rest of CASTEP).

`HS (real)`
:   Height of Slice : Amplitude of the periodic cuts made in the
    crossover stage.

`CP (real)`
:   Cut position: centre of the periodic cut.

`WZ (logical)`
:   Write xyz: whether or not to write the xyz file.

`WX (logical)`
:   Wrap xyz when writing to xyz file or not

`WC (logical)`
:   Cut symbols: label the xyz according to cut (true) or regular
    (false) symbols.

`PC (logical)`
:   Periodic cut: Whether or not to use periodic or planar cuts.

`PR (real)`
:   Permute rate: Probability of a pair of atoms\' positions being
    permuted the mutation step.

`SPR (real)`
:   Spin permute rate: Probability of a pair of atoms\' spins being
    permuted in the mutation step.

`GT (logical)`
:   Transmogrify: Whether or not to try and find equivalent unit cells
    with fewer k-points.

`GC (real)`
:   Constraint percent: Allowed percentage change in atom count for
    variable atom calculations.

`CSO (logical)`
:   Check S Overlap: Check the S overlap matrix on new members to avoid
    atoms being too close together for DFT.

`ICF (logical)`
:   Initial cell fix: fix the cell parameters for generation 0?

`IIA (real)`
:   Initial ion amplitude: Ion mutation amplitude applied to the
    original input cell to generate the first generation.

`ICA (real)`
:   Initial cell amplitude: Cell vector mutation amplitude applied to
    the original input cell to generate the first generation.

`ISA (real)`
:   Initial spin amplitude: Atomic spin mutation amplitude applied to
    the original input cell to generate the first generation.

`IPR (real)`
:   Initial permute rate: Permutation rate applied to the the original
    input cell to generate the first generation.

`ISPR (real)`
:   Initial spin permute rate: Permutation rate applied to the spins of
    the original input cell to generate the first generation.

`MS (integer)`
:   Max simultaneous jobs: Maxmimum number of castep calculations that
    can run simultaneously.

`VKP (logical)`
:   Variable k-points: whether or not to allow variable k-point grids.

## General Inputs - Devel Block - Added in MOGA development, but work with GA

Some of the extra functionality added during development of the MOGA can
also be applied to the GA with the same justification. The following
options, which were added in MOGA development, are also valid GA options
(given within `GA:` and `:ENDGA` devel code block.

`ODD_C`

:   Method used to choose which of two children bred in a single
    breeding operation should be carried forwards into child generation
    if needed:

    `R`
    :   (Default) Randomly choose one from the pair of two.

    `S`
    :   Choose the highest symmetry child. Note this is done before cell
        relaxation.

`BC`

:   \[default `T` \]Use both children bred during a breeding operation.

    `T`
    :   Both children bred during a breeding operation are carried into
        a child population. The only exception is if the number of
        children is odd the last child is chosen based on the `ODD_C`
        option.

    `F`
    :   Only one child per breeding operation is carried into the child
        population. Choice between two children is done using method
        given in `ODD_C`.

`NUM_CHILDREN`
:   \[default `ga_pop_size` \] The number of children per generation.
    Note that the minimum number of converged members per generation
    (`COB` and `MC`) should be set appropriately. Note default (if `RSC`
    not set) will breed `NUM_CHILDREN-1` children and randomly generate
    1 for `NUM_CHILDREN>9`.

`MC`
:   \[default `T` \] Use minimum converged members, i.e. require a
    minimum of 2 converged members in a generation to continue (T/F). If
    `T` and there are under 2 members with converged geometry
    optimisations in a generation the MOGA will exit with an error.

`SS`
:   \[default `F` \] Use the \`snap to symmetry\' method for
    optimisation (T/F). This means geometry optimisation is carried out,
    then snapped to symmetry sites, then relaxed further with a fixed
    cell geometry optimisation.

`RSC`

:   \[default `T` if `NUM_CHILDREN` \>9 else `F` \] Boolean, add
    randomly generated cells from unrepresented (or under represented)
    space groups to child populations before cell relaxation. Note,
    default behaviour (if `RSC` not given) will reduce `NUM_CHILDREN` by
    1 (if `NUM_CHILDREN` \>9) and add 1 random cell to each child
    generation. This will keep size of total children the same as
    `NUM_CHILDREN` which defaults to `ga_pop_size`. I.e. if neither
    `NUM_CHILDREN` or `RSC` is given then
    `size(parents)==size(children)` and if `size(children)>9` each child
    population will contain 1 randomly generated cell. If `RSC` is given
    all default behaviour is ignored, i.e. if `RSC=T` or `RSC=F` then
    `size(children)==NUM_CHILDREN+RSN`.

    `T`
    :   Add random cells to child generations.

    `F`
    :   Default. Do not add any randomly generated cells to any child
        population.

`RSN`
:   \[default `1` \] Add this many randomly generated children of
    unrepresented space group at every generation where children are
    added. Requires `RSC=T` to use.

`RSG`
:   \[default `1` \] Integer: Random Symmetry Generation. Add random
    cells to the child population of generation 1 and then every `RSG`
    generations afterwards. E.g. for `RSG=2` new children will be added
    to child population of generation 1,3,5,... Requires `RSC=T` to use.

`IPM`

:   \[default `M` \] Initial Population Method: The method used for
    initial population generation, where the parent population of
    generation 0 is created from the input `<seed>.cell`.

    `M`
    :   Use large mutation probabilities and amplitudes to mutate the
        input `<seed>.cell` into randomised cells. This is the same
        method used in the GA and the current default.

    `R`
    :   Randomly select space groups for each new cell and generate
        based on the input `<seed>.cell` such that newly generated
        lattices and numbers of ions are related. This results in a
        population containing different space groups with each species
        in `<seed>.cell` represented.

`RSGL`
:   \[Integer, default \~100\~\] Maximum number of times to attempt a
    generation of a random lattice in random high symmetry cell
    generation. Each will have a maximum number of RSGI attempts to
    place ions in the lattice to create a cell of the given space group,
    giving a maximum of RSGL x RSGI attempts to create a random cell of
    a given space group.

`RSGI`
:   \[Integer, default \~10\~\] Maximum number of times to attempt to
    populate a randomly generated lattice with ions during high symmetry
    cell generation. Therefore, maximum attempts at cell generation is
    RSGL x RSGI, with generation exiting as soon as a valid cell is
    found.

`FCGO`
:   \[default \~F\~\] \`Fixed cell geom opt\', if \`T\' then each
    population members geometry optimisation will be a fixed cell geom
    opt. The cell will be allowed to change only during
    mutation/breeding/etc, not in the geom opt. If set to \'F\' this
    wont occur. Note: If \'T\' then fix~allcell~ cannot be given as a
    cell parameter in input cell.

### `NUM_CHILDREN` and Population Size

If `NUM_CHILDREN` is not set then each generation consists of taking a
population of parents of size `ga_pop_size` and breeding the same number
of children from them. This results in a combined parents + children
population of size `2*ga_pop_size`, from which the fitness of each
member is evaluated and the parent population used for the next
generation selected.

If `NUM_CHILDREN` is given then the number of children bred per
generation can be different than the number of parents. In such a case
`NUM_CHILDREN` children will be bred from a parent population (of size
`ga_pop_size`). The union population created of parents + children will
have size `NUM_CHILDREN + ga_pop_size`, from which (after fitness
evaluation) the parent population for the next generation (of size
`ga_pop_size`) will be selected.

Note that the breeding method in the GA breeds pairs of children from
pairs of parents. If the number of children is even then both children
from each breeding operation are added to the child generation. E.g. a
child population size of 12 will consist of 6 pairs of children from 6
breeding operations.

If the number of children used is odd only one child from the final
breeding operation will be chosen for inclusion into the child
population. This is done by selecting the highest symmetry child (before
relaxation), or if both children are members of the same space group
then the selection is carried out randomly.

## Command for Calling Castep (`CMD`)

The default method for running the population cell relaxations is to use
`castep.serial`, however the command used to call CASTEP can be manually
set from within the `CMD` flags which occupy their own line in the devel
code block.

For example for serial calculations (default) the following line can be
added to the devel block

    CMD: castep.serial :ENDCMD

Or for a parallel calculation with 4 MPI task per CASTEP calculation:

    CMD: mpirun -np 4 castep.mpi :ENDCMD

Such that for the devel line `CMD: [COMMAND] :ENDCMD` each cell
relaxation in the GA will be called with `[COMMAND] <seed>` for the
relevant seed for the given population member.

Note that whichever castep command is used, it must be in the `$PATH`.

## Roulette Selection in the GA (`CS`)

Roulette selection is the default method for selection of pairs of
parents for breeding (i.e. the crossover scheme `CS`) and recommended
over the random method.

Fitness based roulette selection is a tried and tested method of
selection for genetic algorithms, where members are chosen for breeding
or continuation onto the next generation using a random selection that
is weighted by their individual fitness.

## Symmetry Methods and `symmetry_generate`

`symmetry_generate` may be employed in any CASTEP calculation to exploit
symmetries within a given cell to reduce number of calculations and
therefore cost of a cell relaxation. This is especially important in a
GA where many relaxations are carried out and structure of cells
generated by the GA is not initially known.

If symmetry based methods are used `symmetry_generate` will markedly
reduce cost over the course of the GA. This is especially true for any
children that are randomly generated for an underrepresented space
group. For this reason it is recommended to always use
`symmetry_generate` unless there is a good reason not to.

## Symmetry Snapping and Symmetry Methods (`SS`)

As a cost reduction method the symmetry snapping method can be enabled
with `SS=T` (default is `F`). This has to be done in conjunction with
enabling symmetry snapping in the base cell file.

The symmetry snap method will run the cell relaxation for a maximum
number of geom opt iterations as given in the input param file. It will
then snap the resultant cell to symmetry sites (within the given
symmetry tolerance) and perform a second relaxation with the same
maximum number of geometry optimisation steps.

Though this does bias the GA to higher symmetry structures (which may or
may not be desirable) it can reduce the computational cost by reducing
total geometry optimisation steps needed for each member.

Note: As the geometry optimisation is carried out twice the actual total
possible geometry optimisation steps used by each member can be up to
twice that given in the base param file.

## Addition of Randomly Generated Children (`RSC`, `RSG` & `RSN`)

The addition of randomly generated cells is done if `RSC=T` is given in
the `GA: :ENDGA` line in the devel code block. Random cells are added
starting with the 1^st^ generation of children, not the randomly
generated initial population of generation 0.

The space group of the relaxed cells found by the GA are recorded at the
end of every generation. The target space group for a randomly generated
child is then selected from this array such that under represented space
groups are explored.

A number of cells (given by `RSN`) are added every i^th^ generation
(with i given by `RSG`) to the child population. These cells are added
before the initial cell relaxation and are treated the same as the other
bred children (except their parent member and generation numbers are all
set to `-1`). As the cells are relaxed after generation the final cells
for which objective function calculation is carried out may not be the
same as the generated cells.

The default (if `RSC` not given) is to add 1 random cell per generation
unless the number of bred children is less than 10, then no children are
added. This is done by reducing the number of bred children by 1. This
is as the default behaviour is such that `size(children)=size(parents)`.
I.e. if `RSC` is not given and `NUM_CHILDREN>9` then the child
generation is made up of `NUM_CHILDREN-1` bred children with one
additional randomly generated cell.

All default behaviour is overridden by explicitly giving the input
argument `RSC=T` or `RSC=F`, so random cells are just appended to the
bred members of the child population. I.e. if `RSC=T` or `RSC=F` is
given as an input then child population size will be `NUM_CHILDREN+RSN`.

Note: Currently only unit cell generation has been implemented. This
method will currently not work for super cell optimisation.

Note: It is possible to run the GA with `NUM_CHILDREN=0` completely
removing breeding operation. To do this `RSC=T` must be given with
`RSG=1` and the desired number of generated cells in a child population
with `RSN`.

### Cell Generation Method

Cell generation is carried out with the `random_cell` module. A lower
limit to the number of atoms in the generated cell is set in the GA as
the minimum number of atoms in any cell in the previous parent
generation. The maximum is set to the maximum number in any cell plus
half the average of the previous parent generation.

A random cell is generated by first generating a real lattice of the
correct lattice type for the given space group. Atoms are then placed in
the cell on Wyckoff sites by randomly selecting from the Wyckoff
multiplicities that if used will not exceed the maximum number of ions.
Random selection biases towards larger multiplicities as these are
generally more common in nature.

This continues until all species are represented and the number of ions
is in between (or equal to) the minimum and maximum bounds. If the
number of atoms exceeds the maximum this attempt fails. The number of
attempts to generate a cell is currently set to 100. If this is exceeded
then the cell generation is considered a failure and a different target
space group is chosen for random cell generation.

Once the cell has been generated it is tested with spglib [^7] to make
sure it is of the correct space group. If it is then the lattice vectors
are adjusted proportionally such that the cell density is equal to the
average density of cells in the parent population.

Note that random cell generation allows for both fixed N and variable N
calculations, however fixed N may greatly restrict the space group cells
that can be generated by the GA. This will generally result in more
cells failing tests (increased cost) but may also result in an error in
some cases. This is because the maximum number of cell generation
attempts is set to 690, so it can try every space group 3 times. This
should be enough, but not if the number of ions in each species cannot
be created from a valid set of multiplicities for some space group.

### Identifying Randomly Generated Members in the Output `.castep` File

To be able to identify population members that were randomly generated
both their parent population generation and member numbers are set to
`-1`. This will be the same for the an initial population generated with
random high symmetry cell generation and any cells generated with a
unrepresented space group throughout the GA.

Note this is similar to how members of the initial population generated
using mutation are identified, with both parent generations of initially
generated members (with mutation) being `-1` but both parent member
numbers being `0`.

## Initial Population Randomisation/Generation (`IPM`)

Generation of the initial population can be carried out two ways, one is
the same as with the GA using random mutation. This takes the input
`<seed>.cell` and applies a large mutation amplitude and probability to
in effect randomise the cell. This means that the parent population in
gen 0 will contain members with the same number of ions as the input
cell, but randomised positions and lattices.

A second method is to generate a set of unique space groups, then
generate cells based on `<seed>.cell` that are of these space groups.
Space group selection selects a different space group for each member of
the parent population of gen 0.

The selected space groups are used to generate cells, with the guide
lattice vector length being the average of a, b and c from
`<seed>.cell`. All species in `<seed>.cell` must be represented and
(depending on if calculation is fixed~N~ or not) the number of ions is
either the same as in `<seed>.cell` or the number in this cell +/- 20%
of number of cells in input cell.

Random generation of initial populations is a well tested method for
both GAs and MOGAs and creation of an initial population with some
biases will inherently introduce some bias to the evolution of the GA.
However, it may be safe to assume (or may even be known) that in some
cases the solution has a high symmetry. In such a case this bias may
reduce cost of a calculation by decreeing convergence time. Furthermore,
high symmetry initial cells can reduce convergence time over completely
random cells, reducing cost even for the same number of population
members/generations evaluated.

## Population Sizes (`NUM_CHILDREN`)

By default each generation the number of children bred is the same as
the number of parents given by `ga_pop_size`. This can be changed by
adding randomly generated cells (see
[*Addition of Randomly Generated Children (\~RSC\~, \~RSG\~ & \~RSN\~)*]{.spurious-link
target="Addition of Randomly Generated Children (~RSC~, ~RSG~ & ~RSN~)"})
or by changing the base size of the child population.

If `NUM_CHILDREN` is not set then each generation consists of taking a
population of parents of size `ga_pop_size` and creating the same number
of children from them (i.e. setting `NUM_CHILDREN=ga_pop_size`. Note
that if `RSC` is not given and `NUM_CHILDREN>9` then `NUM_CHILDREN-1`
children will be bred and 1 child will be randomly generated as a member
of an underrepresented space group. If `RSC` is given then
`NUM_CHILDREN` is the number of children bred in a child population
(with additional random cells given by `RSN` and `RSG`).

If `NUM_CHILDREN` is given the size of a child population can be
different than the number of parents. In such a case `NUM_CHILDREN`
children will be bred from a parent population (of size `ga_pop_size`).
The union population created of parents + children will have size
`NUM_CHILDREN + ga_pop_size`, from which (after fitness evaluation) the
parent population for the next generation (of size `ga_pop_size`) will
be selected. This assumes `RSC=F`.

Note that the breeding method in the GA breeds pairs of children from
pairs of parents. If the number of children is even then both children
from each breeding operation are added to the child generation. E.g. a
child population size of 12 will consist of 6 pairs of children from 6
breeding operations.

If the number of children used is odd only one child from the final
breeding operation will be chosen for inclusion into the child
population. This is done by using the method given with the `ODD_C`
input if given, if not the default random selection is used.

## Child Selection from Bred Pair Of Children (`ODD_C` & `BC`)

The default method of breeding uses periodic slicing to create two
children from a pair of parents that are bred together. By default both
of these children are added to the child population (for relaxation and
objective function calculation). However, if the use of both children is
turned off with `BC=F` only one child per breeding operation is used
(default if not specified is `BC=T`).

If `BC=F` a choice is made between the two children based on the option
given with `ODD_C`. The two options for this are randomly choosing a
child (`R`) or choosing a child with the highest symmetry (before cell
relaxation) and if both are the same then randomly choosing. The default
is to randomly choose, but both methods result in one of the children
found during breeding being discarded.

The method given with `ODD_C` is also used if the number of children per
generation is odd. If this is the case the last breeding operation
results in two children, one of which is chosen to complete the child
population.

# Restarting the GA

In order to restart the GA after either a failed run or to run for more
generations, the `.xyz` file generated by the GA must be present. For a
MOGA run, then the `.MOGAobj` files for all population members (or
specifically all the parents and children of the last generation) must
also be present.

This is because though some information is read from the `.xyz` file the
`.MOGAobj` files are re-read to assign the objective function values for
the restart population.

In order to perform a restart the `continuation` keyword must be given
in the seed `param` file pointing to the relevant `.xyz` file. E.g. if
the seed was `Si_227`, and the directory contains the file `Si_227.xyz`
then the line `continuation=Si_227.xyz` must be added to the
`Si_227.param` file.

Restarting is done from the last completed population, so if the GA
failed part way through a generation it will restart from the beginning
of this generation. E.g. if the GA fails part way through generation 12
and a restart is performed it will restart from the beginning of
generation 12 (having to recalculate all members).

## NOTE: Restarting Generation and Randomness

The restart is performed by reading the final parent and child
population from the last completed generation. This essentially puts the
GA at the point before it selects the parents which will form the
initial population for a generation (from which the children will be
bred and evaluated).

Although the parents chosen from the parent+child population from the
previous generation will be the same, the pairs of parents chosen for
breeding may not be due to the randomness in the selection process. It
follows that the children will also probably differ.

## NOTE: Automatic File Deletion

If the GA is restarted, it will attempt to **delete all the files** used
for the \"next\" generation. This is the generation after the last fully
completed generation. This is because the GA needs a specific file
format for the files it creates/reads from, and having files already
existent and appending to them will break the GA workflow.

E.g. If the GA fails for some reason part way through generation 6 and
it is restarted, all files created for generation 6 will be deleted. The
GA will then restart from the beginning of generation 6.

It is important therefore to back up any files of interest for the last
uncompleted generation if they are of interest.

Note that if a generation completes successfully this is not an issue.
E.g. if the GA was set to run for 6 generations completing without
error, then the user decides to restart to run until generation 12; the
GA will start at generation 7, with no files needing to be deleted.

## Author

Written by Scott Donaldson, V0.1, 27/06/2020

# Footnotes

[^1]: Abraham N L and Probert M I 2006 Physical Review B - Condensed
    Matter and Materials Physics 73 224104 URL
    <https://link.aps.org/doi/10.1103/PhysRevB.73.224104>

[^2]: Abraham N L and Probert M I 2008 Physical Review B - Condensed
    Matter and Materials Physics 77 134117 URL
    <https://link.aps.org/doi/10.1103/PhysRevB.77.134117>

[^3]: Abraham N L : A Genetic Algorithm for Crystal Structure Prediction
    (PhD thesis) <https://www.cmt.york.ac.uk/cmd/nla_thesis.pdf>

[^4]: Higgins E J, Hasnip P J and Probert M I J : Simultaneous
    Prediction of the Magnetic and Crystal Structure of Materials Using
    a Genetic Algorithm. Crystals 9 2073-4352
    <https://doi.org/10.3390/cryst9090439>

[^5]: Higgins E : A Genetic Algorithm for Magnetic Materials Structure
    Prediction (PhD thesis)
    <https://www.cmt.york.ac.uk/cmd/ejh_thesis.pdf>

[^6]: Bauer M, Probert M and Panosetti C : Systematic Comparison of
    Genetic Algorithm and Basin Hopping Approaches to the Global
    Optimization of Si(111) Surface Reconstructions. J Phys Chem A 126
    1089-5639 <https://doi.org/10.1021/acs.jpca.2c00647>

[^7]: "SPGLIB: a software library for crystal symmetry search", Atsushi
    Togo and Isao Tanaka, <https://arxiv.org/abs/1808.01590> (written at
    version 1.10.4)
