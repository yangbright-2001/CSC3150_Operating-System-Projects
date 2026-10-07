# CSC3150: Operating System

Course projects for **CSC3150: Operating System (2022 Fall)** at The Chinese University of Hong Kong, Shenzhen.

This repository contains the source code and reports for four assignments. Each assignment is in its own directory at the repository root.

1. [Assignment 1: Process Control](#1-assignment-1-process-control) — `Assignment_1_120090874`
2. [Assignment 2: Multithreading](#2-assignment-2-multithreading) — `Assignment_2_120090874`
3. [Assignment 3: Virtual Memory on a GPU](#3-assignment-3-virtual-memory-on-a-gpu) — `Assignment_3_120090874`
4. [Assignment 4: A File System on a GPU](#4-assignment-4-a-file-system-on-a-gpu) — `Assignment_4_120090874`

## 1. Assignment 1: Process Control

Implement the same process-creation flow twice, once as a user-space program and once as a kernel module. In both cases, determine how the child terminates and which signal is involved.

### 1.1 Task 1 (user mode)

1. The parent calls `fork`. The child uses `exec` to run a given test program.
2. The parent waits for the child with `wait` / `waitpid`.
3. After the child finishes, the parent prints how it terminated and the corresponding signal.
4. The program must cover the termination cases supplied by the course, including a normal exit, termination by a signal, and a stopped process (`normal`, `abort`, `alarm`, `bus`, `floating`, `hangup`, `illegal_instr`, `interrupt`, `kill`, `pipe`, `quit`, `segment_fault`, `stop`, `terminate`, `trap`).

### 1.2 Task 2 (kernel mode)

1. Repeat the Task 1 flow inside a kernel module: create a child, run the test program, wait, and interpret the returned status.
2. Use kernel interfaces such as `kernel_clone`, `do_execve`, and `do_wait`. Export the required symbols with `EXPORT_SYMBOL` and rebuild the kernel.
3. Load the module with `insmod` and read the child's termination reason and signal from the kernel log (`dmesg`).

## 2. Assignment 2: Multithreading

### 2.1 Main task: Frog Crosses the River

Implement the game "Frog Crosses the River" with multiple threads.

1. Logs float on the river. The frog starts on the near bank, and the player moves it up, down, left, and right from the keyboard.
2. Banks, the frog, and logs are drawn with the specified characters.
3. The frog wins by reaching the far bank. It loses if it falls into the water, or if a log carries it past the left or right edge of the river.
4. Logs on odd and even rows move in opposite directions.
5. Log motion, keyboard input, and screen refresh run on separate threads. The shared map and game state must be protected by a mutex.

### 2.2 Bonus: thread pool

Add a thread pool to the given HTTP server so that it can serve requests concurrently.

1. `async_init(num_threads)` creates a fixed number of worker threads.
2. `async_run(handler, args)` enqueues a task and returns immediately. It must not create any more threads.
3. Idle threads must sleep. Busy waiting is not allowed. A sleeping thread must wake as soon as a new task arrives.

## 3. Assignment 3: Virtual Memory on a GPU

Simulate virtual memory, paging, and page replacement with CUDA GPU memory.

### 3.1 Main task

1. Secondary storage (the disk) is 128KB of global memory. `data.bin` is first loaded into an input buffer of the same size.
2. Physical memory is 48KB of shared memory: 32KB for data and 16KB for the page table.
3. Implement `vm_write` to store data at a virtual address in physical memory, and `vm_read` together with `vm_snapshot` to copy data back into a 128KB result buffer.
4. The page size is 32 bytes. The logical address space is larger than physical memory, so a full physical memory must trigger a page swap.
5. Use an inverted page table, and use LRU to choose the page to evict.
6. Count page faults. For the given `user_program.cu`, the page-fault count should be 8193, and `snapshot.bin` should match `data.bin`.

### 3.2 Bonus

Support the same virtual-memory operations with multiple threads (4 CUDA threads are enough). One accepted approach is to split reads and writes by address, so thread `pid` handles only accesses where `addr % 4 == pid`. The result should match the single-thread run.

## 4. Assignment 4: A File System on a GPU

Simulate a simple file system on one contiguous volume, with open, read, write, remove, and list operations.

### 4.1 Storage layout

1. The volume control block is 4KB and is used as a bitmap of occupied storage blocks.
2. The file-content area is 1024KB. The storage block size is 32 bytes, and files use contiguous allocation.
3. The FCB area is 32KB and holds at most 1024 files. Each file has 32 bytes of metadata (name, valid bit, start block, size, creation time, modification time, and so on).

### 4.2 Required operations

1. `fs_open`: open a file for reading or writing. In write mode, if the file does not exist, allocate an FCB and at least one storage block. The returned file pointer must encode the open mode and the FCB.
2. `fs_read`: read the requested number of bytes from the file.
3. `fs_write`: write the given contents into the file. If there are not enough free blocks immediately after the file, compact the volume first, then write the file into the contiguous free space.
4. `fs_gsys(RM)`: delete a file by name and release its FCB and storage blocks.
5. `fs_gsys(LS_D)`: list files by modification time.
6. `fs_gsys(LS_S)`: list files by size. When two files have the same size, the one created earlier comes first.

### 4.3 Bonus

Extend the flat set of files into a directory tree. Distinguish files from directories, record the parent, the first child, and the next sibling, and support `mkdir` and `fs_gsys` on that directory structure.
