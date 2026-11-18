# Execution Control

| Command | Use when |
| --- | --- |
| `starti` | The agent needs PIE bases, loader state, or pre-relocation state |
| `start` | Inspection should begin at `main` with symbols resolved |
| `run` | The agent needs a complete target run or clean restart |
| `run ARG...` | Arguments must be supplied to a fresh run |
| `b SYMBOL` | Execution should stop at a named function or line |
| `b *0xADDR` | The agent knows the runtime address, PIE target, or gadget address |
| `tbreak LOCATION` | The agent needs one stop without leaving a persistent breakpoint |
| `b LOCATION if EXPR` | A location is hit many times but only one state matters |
| `continue` / `c` | Execution should resume after a stop |
| `next` / `n` | Source-level progress must not enter calls |
| `step` / `s` | A called function must be entered |
| `finish` | The current function should run to return |
| `stepi` / `si` | One machine instruction must be executed, including into calls |
| `nexti` / `ni` | One machine instruction must be executed while stepping over calls |
| `until *0xADDR` | A later instruction in the same frame must be reached |
| `return [VALUE]` | The rest of a function must be skipped |
| `nextcall` | Stop immediately before the next `call` |
| `nextjmp` / `nextjump` | Stop at the next branch |
| `nextret` | Stop at the next return-like instruction |
| `nextsyscall` | A direct target syscall must be reached |
| `nextproginstr` | Resume only after returning from libc/loader to the main binary |
| `stepuntilasm MNEMONIC [OPERANDS...]` | Stop at a concrete assembly form |
| `watch EXPR` | Writes to a value must stop execution |
| `awatch EXPR` | Reads or writes of a value must stop execution |
| `rwatch EXPR` | Only readers of a value must stop execution |
| `catch syscall [NAME]` | A syscall boundary, optionally one syscall, must stop execution |
| `catch signal [NAME]` | A signal delivery/reception boundary must stop execution |
| `info breakpoints` | Existing breakpoints and their numbers/conditions must be verified |
| `info watchpoints` | Existing watchpoints and their numbers/expressions must be verified |
| `info catchpoints` | Existing catchpoints and their numbers must be verified |
| `disable NUMBER` | A stop should remain defined but stop firing |
| `enable NUMBER` | A disabled stop should become active again |
| `delete NUMBER` | An obsolete stop must be removed before reloading setup |

Prefer `starti` before resolving PIE bases or runtime mappings. Prefer conditions and temporary breakpoints over manually continuing through noisy stops.
Verify stop numbers with the relevant `info` command before conditioning, disabling, deleting, or re-sourcing setup. Stop numbers are session-local.
