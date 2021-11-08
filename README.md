# Summary

`elfloader` is a super simple loader for ELF files that generates a flat
in-memory representation of the ELF.

It simply concatenates all `LOAD` sections together, using zero-padding if
there are gaps, into one big flat file.

This file includes zero-initialization of `.bss` sections, and thus can be used
directly as a shellcode payload.

If you don't want to waste time with fail-open linker scripts, this is probably
a great way to go.

This doesn't handle any relocations, it's on you to make sure the original ELF
is based at the address you want it to be at.

# Usage

To use this tool, simply:

`cargo run <input elf> <output FELF>`

# Iternals

This tool doesn't care about anything except for `LOAD` sections. It determines
the endianness (little vs big) and bitness (32 vs 64) from the ELF header,
and from there it creates a flat image based on program header virtual
addresses (where it's loaded), file size (number of initialized bytes) and
mem size (size of actual memory region). The bytes are initialized from the
file based on the offset and file size, and this is then extended with zeros
until mem size (or truncated if mem size is smaller than file size).

These `LOAD` sections are then concatenated together with zero-byte padding
for gaps.

This is designed to be incredibly simple, and agnostic to the ELF input. It
could be an executable, object file, shared object, core dump, etc, doesn't
really care. It'll simply give you the flat representation of the memory,
nothing more.

This allows you to turn any ELF into shellcode, or a simpler file format that
is easier to load in hard-to-reach areas, like embedded devices. Personally,
I developed this for my MIPS NT 4.0 loader which allows me to run Rust code.

# FELF0001 format

This tool by default generates a FELF file format. This is a Falk ELF. This
is a simple file format:

```
FELF0001 - Magic header
entry    - 64-bit little endian integer of the entry point address
base     - 64-bit little endian integer of the base address to load the image
<image>  - Rest of the file is the raw image, to be loaded at `base` and jumped
           into at `entry`
```

