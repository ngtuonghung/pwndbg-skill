# Threads And Forks

Thread focus and scheduler/fork settings affect later commands. Verify or reset them deliberately. Record every changed focus, scheduler-locking, follow-fork, and detach-on-fork setting, and deliberately restore it.

| Command | Use when |
| --- | --- |
| `threads` | The agent needs a compact enhanced view of all threads |
| `threads COUNT` | A long thread list must be narrowed |
| `threads -c` | Listing should obey the configured context limit |
| `info threads` | Prefer plain GDB fallback, LWP details, or exact frames |
| `thread N` | Later commands must apply to another thread |
| `thread apply N COMMAND` | One thread must be inspected without permanently changing focus |
| `thread apply all COMMAND` | The same question must be answered for every thread |
| `break LOCATION thread N` | Other threads at the same code must pass through |
| `break *0xADDR thread N` | A raw runtime address must be scoped to one thread |
| `set scheduler-locking off` | Other threads should continue naturally |
| `set scheduler-locking on` | One thread must be isolated from races |
| `set scheduler-locking step` | Locking applies only during single steps |
| `set non-stop on` | One breakpoint must not freeze every worker/connection |
| `set mi-async on` | GDB needs asynchronous target control for non-stop |
| `set target-async on` | Following older examples that use deprecated spelling |
| `set follow-fork-mode parent` | The listener/parent must remain the debugger target |
| `set follow-fork-mode child` | The newly forked child is the inspection target |
| `show follow-fork-mode` | Active fork-following behavior must be verified |
| `set detach-on-fork on` | The non-followed process should run independently |
| `set detach-on-fork off` | Both fork sides must remain under debugger control |
| `show detach-on-fork` | The agent must verify whether the debugger releases or holds the other side |
| `catch fork` | The fork boundary must be caught deterministically |
| `catch vfork` | The vfork boundary must be caught |
| `catch exec` | A child may replace its image and invalidate parent symbols |
| `info inferiors` | All GDB-controlled processes must be listed |
| `inferior N` | Inspection must move between process states |

Thread IDs must come from an observed existing `threads` or `info threads` result. A pre-run scoped breakpoint can be deleted before the thread exists. In all-stop mode, other threads may be stopped at the event. The trigger, not the post-stop display, identifies the scoped thread. For a forking server, set `follow-fork-mode child`, set `detach-on-fork on`, set `catch fork`, continue, and then inspect the child. After following an `exec` child, restore the target with `file ./TARGET` before restart.
