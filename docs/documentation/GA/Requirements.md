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
