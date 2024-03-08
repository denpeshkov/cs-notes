---
tags:
  - OS_Arch
---

# The Question

What happens during execution of the statement `int x = 10`:

1. [Virtual address](Virtual%20Memory.md) to physical address conversion (TLB lookup)
2. TLB miss
3. TLB update (might involve OS)
4. OS may need to swap in page to get the appropriate page table (load from disk to physical address)
5. [Cache lookup](Cache%20Memory.md) (tag check)
6. Determine line not in cache (need to generate read-miss)
7. Arbitrate for bus
8. Win bus, place address, command on bus
9. All caches [perform snoop](Snoop-Based%20Multiprocessor%20Design.md) (e.g., invalidate their local copies of the relevant line)
10. Another cache or memory decides it must respond (let's assume it's memory)
11. Memory request sent to memory controller
12. Memory controller is itself a scheduler
13. Memory controller checks active row in [DRAM row buffer](Random%20Access%20Memory.md). (May need to activate new DRAM row. Let's assume it does.)
14. DRAM reads values into row buffer
15. Memory arbitrates for data bus
16. Memory wins bus
17. Memory puts data on bus
18. Requesting cache grabs data, updates cache line and tags, moves line into exclusive state
19. Processor is notified data exists
20. Instruction proceeds

This list is certainly not complete

# References

- [CMU 15-418/15-618 Parallel Computer Architecture and Programming](References.md#CMU%2015-418/15-618%20Parallel%20Computer%20Architecture%20and%20Programming)
