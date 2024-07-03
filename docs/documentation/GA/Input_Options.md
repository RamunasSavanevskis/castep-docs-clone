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
[Serial and Parallel Calculations](Outline.md#serial-and-parallel-calculations),
[Requirements](Requirements.md) and
[Command for Calling Castep](Input_Options.md#command-for-calling-castep-cmd), if not given all seeded
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

Additional functionality is available within the "devel block" part of
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
:   Permute rate: Probability of a pair of atoms' positions being
    permuted the mutation step.

`SPR (real)`
:   Spin permute rate: Probability of a pair of atoms' spins being
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
:   [default ```F```] Use the ```snap to symmetry``` method for
    optimisation (T/F). This means geometry optimisation is carried out,
    then snapped to symmetry sites, then relaxed further with a fixed
    cell geometry optimisation.

`RSC`

:   [default ```T``` if `NUM_CHILDREN` \>9 else `F` ], add
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
:   [default ```1``` ] Add this many randomly generated children of
    unrepresented space group at every generation where children are
    added. Requires `RSC=T` to use.

`RSG`
:   [default ```1```] Integer: Random Symmetry Generation. Add random
    cells to the child population of generation 1 and then every `RSG`
    generations afterwards. E.g. for `RSG=2` new children will be added
    to child population of generation 1,3,5,... Requires `RSC=T` to use.

`IPM`

:   [default ```M``` ] Initial Population Method: The method used for
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
:   [Integer, default ```100```] Maximum number of times to attempt a
    generation of a random lattice in random high symmetry cell
    generation. Each will have a maximum number of RSGI attempts to
    place ions in the lattice to create a cell of the given space group,
    giving a maximum of RSGL x RSGI attempts to create a random cell of
    a given space group.

`RSGI`
:   [Integer, default ```10```] Maximum number of times to attempt to
    populate a randomly generated lattice with ions during high symmetry
    cell generation. Therefore, maximum attempts at cell generation is
    RSGL x RSGI, with generation exiting as soon as a valid cell is
    found.

`FCGO`
:   [default ```F```] Fixed cell geom opt, if ```T``` then each population members geometry optimisation will be a fixed cell geom opt. The cell will be allowed to change only during mutation/breeding/etc, not in the geom opt. If set to ```F``` this won't occur. Note: If ```T``` then fix ```allcell``` cannot be given as a cell parameter in input cell.

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

Once the cell has been generated it is tested with spglib^[7]^ to make
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
[Addition of Randomly Generated Children (RSC, RSG & RSN)](Input_Options.md#addition-of-randomly-generated-children-rsc-rsg-rsn))

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
