### 笔记

##### AFL进程间通信

在linux中，使用shm共享内存进行通信。

```c++
shm_id = shmget(IPC_PRIVATE, MAP_SIZE, IPC_CREAT | IPC_EXCL | 0600);
```



###### Linux进程通信

IPC_PRIVATE [ https://blog.csdn.net/niepangu/article/details/55271708 ]：
使用IPC_PRIVATE创建的IPC对象, key值属性为0，和IPC对象的编号就没有了对应关系。这样毫无关系的进程，就不能通过key值来得到IPC对象的编号（因为这种方式创建的IPC对象的key值都是0）。因此，这种方式产生的IPC对象，和无名管道类似，不能用于毫无关系的进程间通信。但也不是一点用处都没有，仍然可以用于有亲缘关系的进程间通信。



[ https://baike.baidu.com/item/shmget/6875075?fr=aladdin ]

用于[Linux](https://baike.baidu.com/item/Linux/27050)进程通信(IPC)共享内存。共享内存函数由shmget、shmat、shmdt、shmctl四个函数组成。

```c++
int shmget(key_t key, size_t size, int shmflg)
```

![1562487174064](C:\Users\username\OneDrive\文档\ResearchLog\assets\1562487174064.png)



### afl-analyze

函数表

| 函数名                | 功能                                                         |
| --------------------- | ------------------------------------------------------------ |
| main                  | 程序入口点                                                   |
| get_qemu_argv         | 修复QEMU模式参数？                                           |
| find_binary           | */\* Find binary. \*/*                                       |
| usage                 | 显示使用说明                                                 |
| detect_file_args      | */\* Detect @@ in args. \*/*                                 |
| setup_signal_handlers | */\* Setup signal handlers, duh. \*/*                        |
| set_up_environment    | */\* Do basic preparations - persistent fds, filenames, etc. \*/* |
| handle_stop_sig       | 处理ctrl+c等类似中止运行操作                                 |
| analyze               | 分析，主要函数                                               |
| dump_hex              | */\* Interpret and report a pattern in the input file. \*/*  |
| show_legend           | */\* Show the legend \*/*                                    |
| show_char             | 显示字符，ASCII 0~32及127~255以16进制（%02x）方式显示        |
| run_target            | */\* Execute target application. Returns exec checksum, or 0 为超时 |
| handle_timeout        | 处理超时信号                                                 |
| write_to_file         | /* Write output file. */                                     |
| read_initial_file     | /* Read initial file. */                                     |
| setup_shm             | /* Configure shared memory. */                               |
| remove_shm            | */\* Get rid of shared memory and temp files (atexit handler). \*/* |
| anything_set          | */\* See if any bytes are set in the bitmap. \*/*            |
| classify_counts       | ？                                                           |





### afl-as

函数表

| 函数名              | 功能                                                         |
| ------------------- | ------------------------------------------------------------ |
| main                | 程序入口点                                                   |
| add_instrumentation | /* Process input file, generate modified_file. Insert instrumentation in all  the appropriate places. */ |
| edit_params         | /* Examine and modify parameters to pass to 'as'. Note that the file name  is always the last parameter passed by GCC, so we exploit this property  to keep the code simple. */ |



### afl-fuzz

函数表（92个函数，真多啊ORZ）

| 函数名                 | 功能                                                         |
| ---------------------- | ------------------------------------------------------------ |
| main                   | 程序入口点                                                   |
| save_cmdline           | /* Make a copy of the current command line. */               |
| get_qemu_argv          | /* Rewrite argv for QEMU. */                                 |
| setup_signal_handlers  | /* Set up signal handlers. More complicated that needs to be, because libc on  Solaris doesn't resume interrupted reads(), sets SA_RESETHAND when you call   siginterrupt(), and does other stupid things. */ |
| detect_file_args       | /* Detect @@ in args. */                                     |
| check_asan_opts        | /* Handle screen resize (SIGWINCH). */                       |
| fix_up_sync            | /* Validate and fix up out_dir and sync_dir when using -S. */ |
| get_core_count         | /* Count the number of logical CPU cores. */                 |
| check_cpu_governor     | /* Check CPU governor. */                                    |
| check_crash_handling   | /* Make sure that core dumps don't go to a program. */       |
| setup_stdio_file       | /* Setup the output file for fuzzed data, if not using -f. */ |
| setup_dirs_fds         | /* Prepare output directories and fds. */                    |
| usage                  | /* Display usage hints. */                                   |
| check_term_size        | /* Check terminal dimensions after resize. */                |
| check_if_tty           | /* Check if we're on TTY. */                                 |
| fix_up_banner          | /* Trim and possibly create a banner for the run. */         |
| check_binary           | /* Do a PATH search and find target binary to see that it exists and isn't a shell script - a common and painful mistake. We also check for a valid ELF header and for evidence of AFL instrumentation. */ |
| handle_timeout         | /* Handle timeout (SIGALRM). */                              |
| handle_skipreq         | /* Handle skip request (SIGUSR1). */                         |
| handle_stop_sig        | /* Handle stop signal (Ctrl-C, etc). */                      |
| sync_fuzzers           | /* Grab interesting test cases from other fuzzers. */        |
| fuzz_one               | 核心函数/* Take the current entry from the queue, fuzz it for a while. This  function is a tad too long... returns 0 if fuzzed successfully, 1 if skipped or bailed out. */ |
| could_be_interest      | /* Last but not least, a similar helper to see if insertion of an  interesting integer is redundant given the insertions done for shorter blen. The last param (check_le) is set if the caller already executed LE insertion for current blen and wants to see if BE variant passed in new_val is unique. */ |
| could_be_arith         | /* Helper function to see if a particular value is reachable through arithmetic operations. Used for similar purposes. */ |
| could_be_bitflip       | /* Helper function to see if a particular change (xor_val = old ^ new) could be a product of deterministic bit flips with the lengths and stepovers attempted by afl-fuzz. This is used to avoid dupes in some of the deterministic fuzzing operations that follow bit flips. We also return 1 if xor_val is zero, which implies that the old and attempted new values are identical and the exec would be a waste of time. */ |
| calculate_score        | /* Calculate case desirability score to adjust the length of havoc fuzzing. A helper function for fuzz_one(). Maybe some of these constants should go into config.h. */ |
| choose_block_len       | /* Helper to choose random block len for block operations in fuzz_one(). Doesn't return zero, provided that max_len is > 0. */ |
| common_fuzz_stuff      | /* Write a modified test case, run program, process results. Handle error conditions, returning 1 if it's time to bail out. This is a helper function for fuzz_one(). */ |
| trim_case              | /* Trim all new test cases to save cycles when doing deterministic checks. The trimmer uses power-of-two increments somewhere between 1/16 and 1/1024 of file size, to keep the stage short and sweet. */ |
| next_p2                | /* Find first power of two greater or equal to val (assuming val under 2^31). */ |
| show_init_stats        | /* Display quick statistics at the end of processing the input directory, plus a bunch of warnings. Some calibration stuff also ended up here, along with several hardcoded constants. Maybe clean up eventually. */ |
| show_stats             | /* A spiffy retro stats screen! This is called every stats_update_freq execve() calls, plus in several other circumstances. */ |
| link_or_copy           | /* Helper function: link() if possible, copy otherwise. */   |
| maybe_delete_out_dir   | /* Delete fuzzer output directory if we recognize it as ours, if the fuzzer is not currently running, and if the last run time isn't too great. */ |
| nuke_resume_dir        | /* Delete the temporary directory used for in-place session resume. */ |
| get_runnable_processes | /* Get the number of runnable processes, with some simple smoothing. */ |
| delete_files           | /* A helper function for maybe_delete_out_dir(), deleting all prefixed files in a directory. */ |
| maybe_update_plot_file | /* Update the plot file if there is a reason to. */          |
| write_stats_file       | /* Update stats file for unattended monitoring. */           |
| find_timeout           | /* The same, but for timeouts. The idea is that when resuming sessions without -t given, we don't want to keep auto-scaling the timeout over and over again to prevent it from growing due to random flukes. */ |
| find_start_position    | /* When resuming, try to find the queue position to start from. This makes sense only when resuming, and when we can find the original fuzzer_stats. */ |
| save_if_interesting    | /* Check if the result of an execve() during routine fuzzing is interesting, save or queue the input test case for further analysis if so. Returns 1 if entry is saved, 0 otherwise. */ |
| write_crash_readme     | /* Write a message accompanying the crash directory :-) */   |
| describe_op            | /* Construct a file name for a new test case, capturing the operation that led to its discovery. Uses a static buffer. */ |
| pivot_inputs           | /* Create hard links for input test cases in the output directory, choosing good names and pivoting accordingly. */ |
| perform_dry_run        | /* Perform dry run of all test cases to confirm that the app is working as expected. This is done only for the initial inputs, and only once. */ |
| check_map_coverage     | /* Examine map coverage. Called once, for first test case. */ |
| calibrate_case         | /* Calibrate a new test case. This is done when processing the input directory to warn about flaky or otherwise problematic test cases early on; and when   new paths are discovered to detect variable behavior and so on. */ |
| write_with_gap         | /* The same, but with an adjustable gap. Used for trimming. */ |
| write_to_testcase      | /* Write modified data to file for testing. If out_file is set, the old file is unlinked and a new one is created. Otherwise, out_fd is rewound and truncated. */ |
| run_target             | /* Execute target application, monitoring for timeouts. Return status information. The called program will update trace_bits[]. */ |
| init_forkserver        | Spin up fork server (instrumented mode only)                 |
| destroy_extras         | /* Destroy extras. */                                        |
| load_auto              | /* Load automatically generated extras. */                   |
| save_auto              | /* Save automatically generated extras. */                   |
| maybe_add_auto         | /* Maybe add automatic extra. */                             |
| memcmp_nocase          | /* Helper function for maybe_add_auto() */                   |
| load_extras            | /* Read extras from the extras directory and sort them by size. */ |
| load_extras_file       | /* Read extras from a file, sort by size. */                 |
| compare_extras_use_d   |                                                              |
| compare_extras_len     | /* Helper function for load_extras. */                       |
| read_testcases         | /* Read all testcases from the input directory, then queue them for testing. Called at startup. */ |
| setup_post             | /* Load postprocessor, if available. */                      |
| setup_shm              | /* Configure shared memory and virgin_bits. This is called at startup. */ |
| cull_queue             | /* The second part of the mechanism discussed above is a routine that goes over top_rated[] entries, and then sequentially grabs winners for previously-unseen bytes (temp_v) and marks them as favored, at least until the next run. The favored entries are given more air time during all fuzzing steps. */ |
| update_bitmap_score    | /* When we bump into a new path, we call this to see if the path appears more "favorable" than any of the existing ones. The purpose of the "favorables" is to have a minimal set of paths that trigger all the bits seen in the bitmap so far, and focus on fuzzing them at the expense of the rest.The first step of the process is to maintain a list of top_rated[] entries for every byte in the bitmap. We win that slot if there is no previous contender, or if the contender has a more favorable speed x size factor. */ |
| minimize_bits          | /* Compact trace bytes into a smaller bitmap. We effectively just drop the count information here. This is called only sporadically, for some new paths. */ |
| remove_shm             | /* Get rid of shared memory (atexit handler). */             |
| classify_counts        | ？/* Destructively classify execution counts in a trace. This is used as a preprocessing step for any newly acquired traces. Called on every exec, must be fast. */ |
| init_count_class16     | ？/* Destructively classify execution counts in a trace. This is used as a preprocessing step for any newly acquired traces. Called on every exec, must be fast. */ |
| simplify_trace         | /* Destructively simplify trace by eliminating hit count information and replacing it with 0x80 or 0x01 depending on whether the tuple is hit or not. Called on every new crash or timeout, should be reasonably fast. */ |
| count_non_255_bytes    | /* Count the number of non-255 bytes set in the bitmap. Used strictly for the status screen, several calls per second or so. */ |
| count_bytes            | /* Count the number of bytes set in the bitmap. Called fairly sporadically, mostly to update the status screen or calibrate and examine confirmed new paths. */ |
| count_bits             | /* Count the number of bits set in the provided bitmap. Used for the status screen several times every second, does not have to be fast. */ |
| has_new_bits           | /* Check if the current execution path brings anything new to the table. Update virgin bits to reflect the finds. Returns 1 if the only change is the hit-count for a particular tuple; 2 if there are new tuples seen.  Updates the map, so subsequent calls will always return 0. This function is called after every exec() on a fairly large buffer, so it needs to be fast. We do this in 32-bit and 64-bit flavors. */ |
| read_bitmap            | /* Read bitmap from file. This is for the -B option again. */ |
| write_bitmap           | /* Write bitmap to file. The bitmap is useful mostly for the secret -B option, to focus a separate fuzzing session on a particular interesting input without rediscovering all the others. */ |
| destroy_queue          | /* Destroy the entire queue. */                              |
| add_to_queue           | /* Append new test case to the queue. */                     |
| mark_as_redundant      | /* Mark / unmark as redundant (edge-only). This is not used for restoring state, but may be useful for post-processing datasets. */ |
| mark_as_variable       | /* Mark as variable. Create symlinks if possible to make it easier to examine the files. */ |
| mark_as_det_done       | /* Mark deterministic checks as done for a particular queue entry. We use the .state file to avoid repeating deterministic fuzzing when resuming aborted scans. */ |
| DTD                    | /* Describe time delta. Returns one static buffer, 34 chars of less. */ |
| DMS                    | /* Describe integer as memory size. */                       |
| DF                     | /* Describe float. Similar to the above, except with a single  static buffer. */ |
| DI                     | /* Describe integer. Uses 12 cyclic static buffers for return values. The value returned should be five characters or less for all the integers we reasonably expect to see. */ |
| locate_diffs           | /* Helper function to compare buffers; returns first and last differing offset. We use this to find reasonable locations for splicing two files. */ |
| bind_to_free_cpu       | #ifdef HAVE_AFFINITY  /* Build a list of processes bound to specific cores. Returns -1 if nothing can be found. Assumes an upper bound of 4k CPUs. */ |
| shuffle_ptrs           | /* Shuffle an array of pointers. Might be slightly biased. */ |
| UR                     | /* Generate a random number (from 0 to limit - 1). This may have slight bias. */ |
| get_cur_time_us        | /* Get unix time in microseconds */                          |
| get_cur_time           | /* Get unix time in milliseconds */                          |



### afl-gcc

函数表

| 函数名      | 功能                                                         |
| ----------- | ------------------------------------------------------------ |
| main        | 程序入口点                                                   |
| edit_params | /* Copy argv to cc_params, making the necessary edits. */    |
| find_as     | /* Try to find our "fake" GNU assembler in AFL_PATH or at the location derived from argv[0]. If that fails, abort. */ |



### afl-gotcpu

函数表

| 函数名             | 功能                                 |
| ------------------ | ------------------------------------ |
| main               | 程序入口点                           |
| measure_preemption | /* Measure preemption rate. */       |
| get_cpu_usage_us   | /* Get CPU usage in microseconds. */ |
| get_cur_time_us    | /* Get unix time in microseconds. */ |



### afl-showmap

函数表

| 函数名                | 功能                                                         |
| --------------------- | ------------------------------------------------------------ |
| main                  | 程序入口点                                                   |
| get_qemu_argv         | /* Fix up argv for QEMU. */                                  |
| find_binary           | /* Find binary. */                                           |
| usage                 | /* Display usage hints. */                                   |
| show_banner           | /* Show banner. */                                           |
| detect_file_args      | /* Detect @@ in args. */                                     |
| setup_signal_handlers | /* Setup signal handlers, duh. */                            |
| set_up_environment    | /* Do basic preparations - persistent fds, filenames, etc. */ |
| handle_stop_sig       | /* Handle Ctrl-C and the like. */                            |
| run_target            | /* Execute target application. */                            |
| handle_timeout        | /* Handle timeout signal. */                                 |
| write_results         | /* Write results. */                                         |
| setup_shm             | /* Configure shared memory. */                               |
| remove_shm            | /* Get rid of shared memory (atexit handler). */             |
| classify_counts       |                                                              |



### afl-tmin

函数表

| 函数名                | 功能                                                         |
| --------------------- | ------------------------------------------------------------ |
| main                  | 程序入口点                                                   |
| read_bitmap           | /* Read mask bitmap from file. This is for the -B option. */ |
| get_qemu_argv         | /* Fix up argv for QEMU. */                                  |
| find_binary           | /* Find binary. */                                           |
| usage                 | /* Display usage hints. */                                   |
| detect_file_args      | /* Detect @@ in args. */                                     |
| setup_signal_handlers | /* Setup signal handlers, duh. */                            |
| set_up_environment    | /* Do basic preparations - persistent fds, filenames, etc. */ |
| handle_stop_sig       | /* Handle Ctrl-C and the like. */                            |
| minimize              | !!/* Actually minimize! */                                   |
| next_p2               | /* Find first power of two greater or equal to val. */       |
| run_target            | /* Execute target application. Returns 0 if the changes are a dud, or 1 if they should be kept. */ |
| handle_timeout        | /* Handle timeout signal. */                                 |
| write_to_file         | /* Write output file. */                                     |
| read_initial_file     | /* Read initial file. */                                     |
| setup_shm             | /* Configure shared memory. */                               |
| remove_shm            | /* Get rid of shared memory and temp files (atexit handler). */ |
| anything_set          | /* See if any bytes are set in the bitmap. */                |
| apply_mask            | /* Apply mask to classified bitmap (if set). */              |
| classify_counts       |                                                              |



