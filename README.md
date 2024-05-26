## Fork of the xv6 kernel with shared memory capabilities

The code for shared memory management is primarily located in `shm.c` and `shm.h`. Similarly, for semaphores, the code is in `sem.c` and `sem.h`.

### Key Points of Modification:
- In `argptr` (`syscall.c`), a check was added to handle the case where the address is in the shared memory space. `argptr` is used in `sysproc.c` for both `shmget` and `shmrem`, as `sh_key_t` is simply a pointer to `sh_key_tt` used by the kernel.
- In `allocproc` (`proc.c`), the bitmap for free shared pages positions is initialized to 0. In `fork`, the `shmcopy` function is called to map the parent's shared pages to the same positions in the child. In `wait`, when an exited child is found, `shmfree` is called to free any shared pages that only this process had.

### Kernel Structures:
- The key for each page is a `char[16]`. There is a `shmtable` structure protected by a spinlock that, for each page, keeps track of whether it is in use, the address returned by `kalloc`, its key, how many processes share it, which processes share it, and where in the shared space they share it. For each semaphore, there is only its value and a spinlock protecting it. Finally, each process has a bitmap (`char[4]`) indicating where mappings exist for it.

More details, explanations, and comments can be found in the corresponding source files.

### Workloads:
- **work1**:
  Forks a child that writes its PID to a shared buffer. The parent then reads the correct data from the buffer. If the child performs `shmrem` and then tries to write, a trap 14 occurs as expected since the mapping for the child no longer exists.
- **work2**:
  Forks 15 processes that each increment one of the 1024 values in each of the 32 shared pages. The master (the one that did the fork) calculates the sum. Using semaphores, the sum is correct, whereas without them, a race condition is observed.
- **work3**:
  Implements the bounded buffer problem using the solution with 3 semaphores as described in "Operating System Concepts, 9th Edition". The program runs continuously, with consumers trying to write and producers trying to read from a buffer with 30 shared pages. Without semaphores, a race condition is observed here as well.
- **work4**:
  Performs various `shmget`, `shmrem`, and `fork` operations to test extreme conditions (e.g., attempting to `shmrem` a page that does not exist).
