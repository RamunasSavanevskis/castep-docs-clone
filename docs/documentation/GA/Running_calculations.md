** Serial and Parallel Calculations

The ~castep_GA~ is a serial program, and must be ran in serial in order to allow for seeding further CASTEP calls. These CASTEP calls can either be synchronous (i.e. the first must finish before the next one can start) or asynchronous (multiple CASTEP jobs running at the same). In addition, each CASTEP job itself may be either serial or parallel (with the usual range of parallelization schemes available). This can be controlled by the user, depending on the size of the computing facility and the number of population members required.

The command used to call CASTEP for the cell relaxations is given in a devel code block in between the flags ~CMD:~ and ~:ENDCMD~, though default is ~castep.serial~ parralelised methods such as ~mpirun castep.mpi~ can also be used. See [[Command for Calling Castep (~CMD~)]].


* Requirements

A working version of CASTEP is required, that will be called with the command given in-between the ~CMD:~ and ~:ENDCMD~ flags in the devel code block. For example with ~CMD: castep.serial :ENDCMD~ or ~CMD: mpirun castep.pmi :ENDCMD~.

The command line tools sed, grep and awk must be installed.


** Param File Requirements

The GA must be enabled using the ~.param~ file option ~task = genetic algor~ and the ~GA=T~ flag set in the ~GA: :ENDGA~ devel code block.

As the GA will edit the base ~.param~ file as part of the spawning CASTEP jobs, the ~.param~ file is required to end with one empty line.


** Compiling ~castep_GA~

Compiling can now be automatically carried out simply by calling 'make tools' from the CASTEP top-level directory.

** License
The GA is built using many standard CASTEP source code routines, and so is distributed as part of the CASTEP source. Consequently, it is covered by the same CASTEP licence terms and is NOT for general distribution.
