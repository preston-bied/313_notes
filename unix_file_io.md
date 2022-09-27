# UNIX File I/O

## I/O

I/O is the process of copying data between the main memory and external devices (disk drives, terminals, networks, etc.).

Language run-time systems provide higher-level facilities for performing I/O (e.g., `printf`/`cout`, `scanf`/`cin`).
- On UNIX systems, these higher-level I/O functions eventually call *System-Level UNIX I/O functions* provided by the Kernel (e.g., `printf()` internally calls `write()`)
- The same applies to Windows and other OS's
- `printf()` calls native Windows function `WriteFile()`

## UNIX Files

A UNIX file is a sequence of m bytes B<sub>0</sub>