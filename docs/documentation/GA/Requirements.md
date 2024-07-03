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
