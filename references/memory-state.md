# Memory And CPU/Stack State

| Command | Use when |
| --- | --- |
| `regs` | The agent needs a concise view of all general-purpose CPU state |
| `regs REG...` | Only selected registers are relevant |
| `info registers [REG...]` | Prefer plain GDB output or a non-pwndbg fallback |
| `set $REG = EXPR` | A general register must be changed or tested |
| `set $pc = 0xADDR` | Execution must be redirected without editing code |
| `x/Nxg EXPR` | The agent needs exact eight-byte words without recursive dereference |
| `x/s 0xADDR` | The agent needs a null-terminated string at a known address |
| `x/Ni $pc` | The agent needs exact instructions around PC |
| `telescope [EXPR]` | Pointer-heavy memory must be understood recursively |
| `telescope EXPR COUNT` | The agent needs a larger or smaller telescope window |
| `tele 0xADDR -r` | A pointer source before the current address must be scanned |
| `stack` | The agent needs pointer-aware context around `$sp` |
| `stack COUNT` | The agent needs a wider stack window |
| `bt` | The call path must be reconstructed |
| `frame N` | Another call frame must become current |
| `up` | Move to the next older frame |
| `down` | Move to the next newer frame |
| `info args` | The agent needs current frame arguments |
| `info locals` | The agent needs current frame locals |
| `canary` | Stack-overflow analysis needs the canary |
| `context` | The current dashboard must be refreshed |
| `context code` | The agent needs only source/code context |
| `set context-sections SECTIONS...` | Default dashboard is noisy or needs fixed order |

Prefer a narrow register or memory view once the agent knows the decisive field.
