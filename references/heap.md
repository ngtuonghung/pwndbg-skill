# Glibc Heap

Stop after the target creates the allocator state and allocation/free history under investigation.

| Command | Use when |
| --- | --- |
| `heap` | Chunk boundaries, sizes, flags, and bin membership must be walked |
| `vis-heap-chunks` | The agent needs a compact visual chunk map |
| `bins` | The agent needs one summary across every glibc bin class |
| `fastbins` | The agent needs single-linked fastbin state |
| `tcachebins` | The agent needs glibc tcache counts and freelists |
| `unsortedbin` | The agent needs recently freed large chunks or consolidation candidates |
| `smallbins` | Sorted small-chunk state must be checked |
| `largebins` | Sorted large-chunk state must be checked |
| `malloc-chunk 0xADDR` | One chunk header must be interpreted |
| `top-chunk` | The agent needs wilderness boundary or available top size |
| `arena` | The agent needs `malloc_state` fields such as bins and thread state |
| `mp` | The agent needs allocator tuning such as thresholds and tcache limits |
