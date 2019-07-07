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

函数表

| 函数名 | 功能 |
| ------ | ---- |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |



### afl-gcc

函数表

| 函数名 | 功能 |
| ------ | ---- |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |



### afl-gotcpu

函数表

| 函数名 | 功能 |
| ------ | ---- |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |



### afl-showmap

函数表

| 函数名 | 功能 |
| ------ | ---- |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |



### afl-tmin

函数表

| 函数名 | 功能 |
| ------ | ---- |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |
|        |      |



