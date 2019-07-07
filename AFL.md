AFL进程间通信

在linux中，使用shm共享内存进行通信。
shm_id = shmget(IPC_PRIVATE, MAP_SIZE, IPC_CREAT | IPC_EXCL | 0600);

IPC_PRIVATE[https://blog.csdn.net/niepangu/article/details/55271708]：
使用IPC_PRIVATE创建的IPC对象, key值属性为0，和IPC对象的编号就没有了对应关系。这样毫无关系的进程，就不能通过key值来得到IPC对象的编号（因为这种方式创建的IPC对象的key值都是0）。因此，这种方式产生的IPC对象，和无名管道类似，不能用于毫无关系的进程间通信。但也不是一点用处都没有，仍然可以用于有亲缘关系的进程间通信。

