# Search, Offset Discovery, Patching, And Extraction

Prefer read-only inspection first. Patch or write only disposable or explicitly approved targets. Capture before/after evidence. Use task-specific output paths. Apply explicit authority rules to non-disposable state.

| Command | Use when |
| --- | --- |
| `search STRING` | A string or byte sequence must be located |
| `search -x -t bytes HEX` | Raw hex must be searched as bytes, not ASCII |
| `search -t pointer 0xADDR` | The agent needs references to a pointer-sized value |
| `search -w STRING` | Only writable data can be changed or corrupted |
| `search VALUE MAPPING` | Search scope must be limited to an executable/library |
| `cyclic COUNT FILE` | Input must carry a unique-position marker |
| `cyclic -l VALUE` | A PC, register, or stack word must become an input offset |
| `patch 0xADDR INSTRUCTION` | Code or a gadget must be changed without recompiling |
| `set {unsigned char}0xADDR = BYTE` | Exact raw bytes must be written |
| `set {int}0xADDR = VALUE` | A typed integer must be overwritten |
| `set {char[N]}EXPR = "BYTES"` | A writable buffer must contain raw string bytes |
| `dump memory FILE 0xSTART 0xEND` | Runtime data must be extracted for offline analysis |
| `dump binary memory FILE 0xSTART 0xEND` | Prefer GDB's explicit binary-dump form |
