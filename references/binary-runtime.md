# Binary And Runtime Layout

| Command | Use when |
| --- | --- |
| `version` | The agent needs exact pwndbg/GDB environment |
| `checksec` | Hardening status affects exploit feasibility |
| `vmmap` | The agent needs runtime mappings, permissions, sizes, or backing files |
| `vmmap NAME` | Only one object's mappings are relevant |
| `piebase` | The agent needs a PIE executable's runtime base |
| `$rebase(0xOFFSET)` | A static offset must become a runtime expression |
| `info symbol 0xADDR` | A raw address must be attributed to a symbol |
| `info functions REGEX` | Function names or declarations must be searched |
| `info address SYMBOL` | The agent needs a symbol's runtime address |
| `xinfo 0xADDR` | Mapping, file/segment offsets, and sections must be seen together |
| `disassemble SYMBOL` | The agent needs a complete function's instructions |
| `nearpc` | Prefer PC-centered annotated disassembly |
| `got` | The agent needs writable GOT or overwrite surface |
| `got -r` | Full RELRO hides the entries being sought |
| `plt` | The agent needs imported stubs or callable PLT addresses |
| `info proc mappings` | Prefer plain OS-style mappings over `vmmap` |
| `info proc exe` | The agent needs the exact running executable path |
| `auxv` | The agent needs runtime ELF/auxv facts |
| `procinfo` | The agent needs PID, cmdline, cwd, identities, or descriptors |
| `attachp PID` | A running target cannot be restarted |
| `gcore FILE` | A live frozen state must be preserved |
| `gdb BINARY CORE` | A saved core must be analyzed with its executable |
| `core-file CORE` | A core must be loaded into an already selected executable |

Use `/usr/bin/gdb-multiarch` for cross-architecture targets. Native `/usr/bin/gdb` supports only x86-64. Verify each pwndbg command against the target architecture. Core state supports inspection, not continue/step. For remote or QEMU user-mode targets, use [remote-multiarch.md](remote-multiarch.md).
