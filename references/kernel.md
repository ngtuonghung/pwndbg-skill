# Linux Kernel, SLUB, And Segment/TLS State

Kernel commands below target pwndbg's current command surface. The slab family supports SLUB only, not SLAB or SLOB. Kernel targets are typically a `qemu -s -S` GDBstub or an equivalent kernel debugging session. Verify kernel-specific syntax against the running pwndbg surface.

## SLUB

| Command | Use when |
| --- | --- |
| `slab list` | The agent needs all active SLUB caches and basic statistics |
| `slab list FILTER` | The agent needs only caches whose names contain a substring |
| `slab info CACHE` | The agent needs one cache's object size, slab, free, and active counts |
| `slab info CACHE -v` | The agent needs a per-slab breakdown of one cache |
| `slab info CACHE -c CPU` | The agent needs a specific per-CPU cache view |
| `slab info CACHE -n NODE` | The agent needs a specific NUMA-node freelist view |
| `slab contains ADDR` | The agent needs the owner cache of an arbitrary address |

## Buddy allocator

| Command | Use when |
| --- | --- |
| `buddydump` | The agent needs page allocator free lists by order |

## Kernel context

| Command | Use when |
| --- | --- |
| `kbase` | The agent needs the runtime kernel virtual base despite KASLR |
| `kchecksec` | The agent needs kernel hardening posture |
| `kconfig CONFIG` | The agent needs a running-kernel build option |
| `kcmdline` | The agent needs the running kernel command line |
| `kversion` | The agent needs the running kernel version |
| `kcurrent` | The agent needs the kernel task currently used by pwndbg |
| `p2v ADDR` | A physical address must become its kernel virtual address |
| `v2p ADDR` | A virtual address must be mapped to its physmap address |
| `pagewalk ADDR` | The agent needs page-table entries backing a virtual address |
| `pageinfo PAGE` | A `struct page *` must be related to a virtual address |
| `msr MSR` | A model-specific register must be read |
| `msr MSR -w VALUE` | A model-specific register must be written |

## x86-64 FS/GS and TLS

| Command | Use when |
| --- | --- |
| `fsbase` | The agent needs the current thread's FSBASE/TCB location |
| `gsbase` | The agent needs the current GS base, often kernel per-CPU data |
| `p/x $fsbase` | FS base must be used inside an expression |
| `p/x $gsbase` | GS base must be used inside an expression |
| `x/gx $fsbase+0x28` | The glibc stack canary must be read manually from the TCB |
| `tls` | The agent needs TLS layout for the current thread |
| `tls -a` | The agent needs the complete TLS dump |
| `tls -p` | TLS should be obtained by calling `pthread_self()` |
| `info registers fs_base gs_base` | Use raw GDB when pwndbg FS/GS commands are unavailable |

For a kernel or `qemu-system` GDBstub, use this reference instead of [remote-multiarch.md](remote-multiarch.md).
