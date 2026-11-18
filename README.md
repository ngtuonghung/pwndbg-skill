# pwndbg-skill

Agent skill for dynamic binary analysis with GDB/pwndbg: live ELF processes, cores, remote targets, and Linux kernels — driven through a stateful two-pane tmux session so runtime state survives across tool calls.

See [SKILL.md](SKILL.md) for the full workflow.

## Core rules

- Pane `0` is the target terminal, pane `1` is pwndbg/GDB. One deliberate interaction per tool call: send, then capture and wait for decisive evidence.
- `send-keys` returning is not completion. Never infer success from silence.
- Treat PIE bases, runtime addresses, mappings, breakpoints, focus, and patches as volatile. Verify with `context`, `bt`, `vmmap`, `info breakpoints`.
- On a crash, capture the still-stopped state (`bt`, `vmmap`, `generate-core-file`) before rerunning or cleaning up.
- Stop every tmux session you create.

## References

Read only the one that answers the current question.

| Reference | Use when |
| --- | --- |
| [execution-control](references/execution-control.md) | Start/restart, stops, stepping, watchpoints, catchpoints |
| [memory-state](references/memory-state.md) | CPU, stack, frames, canary, dashboard |
| [binary-runtime](references/binary-runtime.md) | Loaded image, address space, process facts, attach, cores |
| [remote-multiarch](references/remote-multiarch.md) | GDB remote protocol, non-x86-64 user-mode targets |
| [heap](references/heap.md) | glibc allocator state |
| [search-patch-dump](references/search-patch-dump.md) | Locate, patch, or extract runtime data |
| [kernel](references/kernel.md) | Linux kernel, QEMU gdbstub, SLUB, page tables, FS/GS, TLS |
| [threads-forks](references/threads-forks.md) | Threads, forks, execs, inferiors |
