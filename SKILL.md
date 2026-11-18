---
name: pwndbg-debugging
description: "Use GDB/pwndbg for live ELF, process, and core analysis through stateful tmux debugging. Cover execution control and CPU, stack, frame, and canary state. Cover memory, binary/runtime layout, remote GDB/multiarch, heap, search/patch/dump, threads/forks/inferiors, kernel context, SLUB, and page tables. Cover reusable `.gdb` setup. Preserve volatile runtime and patch state. Choose the narrowest reference."
---

# Pwndbg Debugging

Use this skill for dynamic binary analysis when GDB/pwndbg fits the task. Keep the target in the least state needed. Prefer pwndbg primitives over hand parsing. Treat PIE, runtime addresses, mappings, breakpoints, and patches as volatile. Live, stateful debugging uses detached tmux. Do not replace it with one-shot pipes or a whole-session script.

## Mandatory tmux model

Use tmux when the target needs terminal I/O. Also use tmux when the target is long-running, stateful, curses-based, or must remain stopped/live while you inspect evidence. Use one deliberate interaction per tool call. Observe, decide, type, and then observe again.

- Pane `0` is the target terminal.
- Pane `1` is pwndbg/GDB unless a documented fallback says otherwise.

Remote does not imply two panes. Use the split only when the remote target needs foreground terminal I/O. A headless `gdbserver` or QEMU stub may run in the background or as an existing daemon while GDB uses one pane. See [remote-multiarch.md](references/remote-multiarch.md). Send command text literally, and then send Return separately:

```bash
tmux send-keys -t SESSION:0.PANE -l 'TEXT'
tmux send-keys -t SESSION:0.PANE C-m
tmux capture-pane -p -t SESSION:0.PANE -S -COUNT
```

`send-keys` returning is not completion. Capture enough history, including scrolled history after dashboard repaints, and wait for a prompt, stop, register dump, error, or other decisive evidence. Verify pane and process state before reuse because `pane_dead`, `pane_current_command`, `pane_tty`, and the target PID/state/TTY can all change. Use a unique task-specific session name. A tmux session owns debugger/process state. Do not blindly recreate, overwrite, or kill a same-named session. Track every live session you create. Before starting a replacement, stop the old disposable session unless the task genuinely requires parallel sessions. Before finishing, stop every owned session.

## Create and attach a standard session

```bash
tmux new-session -d -s pwndbg-task -x 220 -y 50
tmux split-window -h -t pwndbg-task:0
tmux select-pane -t pwndbg-task:0.0
```

Start an interactive target as an ordinary child in pane 0. Never use `exec` because it can end the pane and shift pane indexes.

```bash
tmux send-keys -t pwndbg-task:0.0 -l '/path/to/target --options'
tmux send-keys -t pwndbg-task:0.0 C-m
tmux display-message -p -t pwndbg-task:0.0 '#{pane_tty}'
pgrep -af '/path/to/target'
ps -o pid=,ppid=,stat=,tty=,args= -p PID
tmux send-keys -t pwndbg-task:0.1 -l 'gdb -q /path/to/target -p PID'
tmux send-keys -t pwndbg-task:0.1 C-m
```

Wait for attach and `pwndbg>`. Pane 0 remains the target's foreground terminal. Target prompts/stdin go to pane 0, and debugger commands go to pane 1. Configure breakpoints/catchpoints before continuing.

If GDB denies attach, preserve the target. Inspect Yama `ptrace_scope`, ancestry, `CAP_SYS_PTRACE`, seccomp, and container policy. A fallback can start from GDB with `set inferior-tty /dev/pts/N`, `set args`, and `starti`/`run`. Use the fallback only when typed terminal interaction is not required, because pane input can be consumed by pane 0's shell. Use it for output-only targets or input supplied by arguments/files/pipes.

For a pre-existing process with irreplaceable state, verify ownership and TTY first. Then attach without restarting it.

## Reusable `.gdb` setup

When setup is nontrivial or reproducibility matters, write `.gdb` files in the task artifact directory. Setup files select symbols/conditions, breakpoints, catchpoints, conveniences, and formats without experiment input or continuation. Put execution-specific `continue`, dumps, and trigger state in a separate focused file. For example:

```gdb
set pagination off
set confirm off
break exit
catch syscall read
condition $bpnum $rdi == 0
info breakpoints
```

Source setup only after GDB selects or attaches to the target and shows `pwndbg>`:

```bash
tmux send-keys -t pwndbg-task:0.1 -l 'source /challenge/chall/artifacts/task-setup.gdb'
tmux send-keys -t pwndbg-task:0.1 C-m
tmux capture-pane -p -t pwndbg-task:0.1 -S -160
```

Use absolute paths, and verify every command in captured output. `info breakpoints` and `info catchpoints` are the oracle. Absence of visible text is not success. A failed source is not success. Do not blindly re-source a file. After verifying them, delete only the exact old breakpoint/catchpoint numbers, and then source the corrected file. For a normal retry, source once in the new GDB session. `$bpnum` avoids hard-coded breakpoint numbers. Syscall/register conditions are architecture-specific.

## Interactive control and special keys

Substitute any pwndbg/GDB command for `TEXT` in the generic send/capture pattern. Focus updates through `thread`, `frame`, `up`/`down`, and `inferior` persist until changed. Treat writes, patches, dumps, core generation, and `!shell` commands as side effects. Capture before/after evidence, and use task-specific output paths.

Without `-l`, tmux interprets key names. `C-c` sends SIGINT/interrupt. `C-z` sends terminal stop. `C-\` sends SIGQUIT. `C-d` may end stdin/shell. `Escape`, `Tab`, `BSpace`, `DC`, `PageUp`, and `PageDown` are useful terminal keys. `C-s` can freeze output, and `C-q` recovers from output freeze. Repeat a key with `tmux send-keys -t PANE -N 3 Escape`.

Decide pane ownership before an interrupt. For an attached interactive target, send `C-c` to pane 0. For a GDB-started fallback, send it to pane 1. Program arguments/input belong to pane 0 only when the inferior actually waits and owns that terminal. A stopped syscall is strong evidence. During cleanup, delete queued target input so the pane shell cannot execute it later.

After any prompt, suspend, context switch, fork/exec, or execution-state transition, capture until the next `pwndbg>` or documented stop. If GDB asks a question, inspect `info inferiors`, `info threads`, or the pending operation before answering. Never infer success from silence.

## Crash, retry, and cleanup

On a fatal signal or unexpected exit, do not immediately rerun. Capture both panes with enough history. Inspect the still-stopped inferior, and preserve the faulting state while it exists:

```gdb
context
bt
vmmap
generate-core-file /task/evidence/target.core
```

Use a new versioned path for each core capture. Verify writes, and do not let repeated retries silently overwrite evidence. Capture only decision-relevant cores, then curate or delete disposable duplicates with the task's evidence policy. If the target exited normally, no faulting state exists. Record the final output. The target exiting does not clean up GDB/tmux. Either retry from the same debugger or perform cleanup.

Retry according to ownership:

- Debugger-started disposable target: `kill` and then `run` preserves symbols, breakpoints, arguments, environment, and GDB's inferior TTY. Skip `kill` if GDB says the target already exited.
- Pane-0 interactive attached target: do not use GDB `run` because it loses the pane's foreground terminal relationship. Capture evidence, kill the dead inferior, restart it as an ordinary pane-0 child, verify the new PID/TTY, and attach to `NEW_PID` from the existing GDB session.
- Materially different binary/args/environment/TTY/hypothesis: use a new session rather than overwrite old evidence.

Before cleanup, capture final state and inspect the session/panes/inferiors. At a GDB prompt, `detach` preserves an externally valuable target. `kill`/`quit` suits a disposable target. Detaching leaves the target/session owned by the task. Record it and clean it later. Do not treat detach as process cleanup. If GDB is hung, send `C-c` to pane 1, wait for a prompt, and then quit. Kill tmux only after evidence capture and only when its panes/processes are disposable. Avoid broad `pkill`, and use PIDs verified by GDB, pane ownership, or task-specific ownership verification. Verify the session is gone before recreating it:

```bash
tmux has-session -t pwndbg-task 2>/dev/null || true
tmux list-panes -t pwndbg-task -F '#{pane_index} #{pane_current_command} #{pane_dead} #{pane_tty}'
```

## References

Read only the reference that answers the current question. Each reference is a command-selection aid. It does not replace the mandatory observe–decide–type loop. Use [execution-control.md](references/execution-control.md) to select start/restart, stops, stepping, watchpoints, and catchpoints. Use [memory-state.md](references/memory-state.md) to inspect stopped CPU, stack, frame, canary, and dashboard state. Use [binary-runtime.md](references/binary-runtime.md) for the loaded image, address space, process facts, attach, and core inspection. Use [remote-multiarch.md](references/remote-multiarch.md) for GDB remote protocol and non-x86-64 user-mode targets. Use [heap.md](references/heap.md) for glibc allocator state, not merely a pointer into the heap. Use [search-patch-dump.md](references/search-patch-dump.md) to locate, patch, or extract runtime data. Use [kernel.md](references/kernel.md) for a Linux kernel or QEMU GDBstub, SLUB, page tables, FS/GS, and TLS. Use [threads-forks.md](references/threads-forks.md) when threads, forks, execs, or inferiors can receive the next event. For live debugging, this mandatory tmux workflow takes precedence over optional reference examples.
