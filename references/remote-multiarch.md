# Remote GDB And Multiarch

Choose the client from the target, not from the launch style:

| Situation | Server | Client | Execution control |
| --- | --- | --- | --- |
| Local x86-64 process | none | `gdb` | `run` |
| Local host process with multiarch client | none | `gdb-multiarch` | `run` |
| Foreign-architecture ELF only for inspection | none | `gdb-multiarch` | symbols/files only, cannot run it |
| Native remote process | `gdbserver` | `gdb` only when it matches the target, otherwise `gdb-multiarch` | `target extended-remote` |
| Foreign process under local QEMU user mode | `qemu-aarch64 -g` / `qemu-arm -g` | `gdb-multiarch` | `target remote` |

Never use native `gdb` for a foreign ELF. It can silently report the host architecture. Select the executable with an absolute path so GDB's working directory cannot differ from the server's.

## Pane layout

Remote does not imply two panes. Use two panes only when the remote target needs typed stdin or must own a foreground terminal. Put the server in pane 0 and GDB in pane 1. A headless/output-only target can run `gdbserver ... &` in one pane followed by GDB. An already running remote endpoint can be attached without creating a server pane. For disposable, noninteractive evidence collection, a bounded batch run is acceptable. Do not use batch mode to preserve or interact with a live/stateful target.

## Native x86-64 remote target

```bash
# Server pane/background process
gdbserver [--once] HOST:PORT /absolute/path/to/target [ARGS...]

# GDB pane/terminal
gdb-multiarch -q /absolute/path/to/target
set architecture i386:x86-64
set sysroot /
target extended-remote HOST:PORT
```

Without `set sysroot /`, GDB may transfer remote libraries under `target:` paths. That is slower and makes symbol provenance harder to reason about. This same-host `/` sysroot is safe only when the debugger can resolve the target's libraries identically. For a distinct remote filesystem, use a matching copy or sysroot. `--once` is useful for a disposable server that exits after one connection.

## QEMU user-mode multiarch target

AArch64:

```bash
# Stub pane/background process
qemu-aarch64 -g PORT -L /usr/aarch64-linux-gnu /absolute/path/to/target

# GDB pane/terminal
gdb-multiarch -q /absolute/path/to/target
set architecture aarch64
set sysroot /usr/aarch64-linux-gnu
target remote HOST:PORT
```

ARMHF:

```bash
# Stub pane/background process
qemu-arm -g PORT -L /usr/arm-linux-gnueabihf /absolute/path/to/target

# GDB pane/terminal
gdb-multiarch -q /absolute/path/to/target
set architecture arm
set sysroot /usr/arm-linux-gnueabihf
target remote HOST:PORT
```

`-L` supplies QEMU's guest loader/libc prefix. `set sysroot` supplies GDB's local symbol path. Both are normally needed for a dynamic guest binary. QEMU user-mode accepts `-g PORT` but not the system-mode `-S` option. It waits for the debugger before executing guest code.

## Remote control and evidence

Set breakpoints after connecting and before continuing. A QEMU user connection normally stops in the guest dynamic loader, not at the executable entry. Verify state before trusting a dashboard:

| Command | Evidence |
| --- | --- |
| `show architecture` | GDB's selected architecture |
| `info inferiors` | remote endpoint and guest PID |
| `info threads` | stopped thread and frame |
| `break FUNCTION` then `continue` | source-level remote execution control |
| `print VARIABLE` | symbol/type resolution |
| `info symbol $pc` | PC attribution |
| `vmmap` / `info proc mappings` | guest mappings and permissions |
| `piebase` / `xinfo $pc` | PIE base and file-offset attribution |

PIE bases are volatile and often differ from native runs under QEMU user emulation. Pwndbg can annotate AArch64 and ARM/Thumb disassembly, but verify the architecture/mode label and `show architecture` after every stop. `procinfo` may explicitly report the QEMU host process. Treat host PID, descriptors, and mappings as inaccurate for guest process state.

For externally valuable state, `detach` resumes the target and ends the GDB connection, but it does not stop a separate `gdbserver`. Stop an owned server afterward. For a disposable QEMU user target, `kill` and then `quit` is deterministic. Capture final evidence first, and then verify that no owned `gdbserver`/`qemu-*` process remains.

For a kernel or `qemu-system` GDBstub, use [kernel.md](kernel.md) instead.
