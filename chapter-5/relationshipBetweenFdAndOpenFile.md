# 文件描述符表和打开文件之间的关系

内核维护了三个数据结构：

- 进程级的文件描述符( open file descriptor )表。该表的每一条目都记录了单个文件描述系统的相关信息。如下所示

    - 控制文件描述符操作的一组表示。根据书中所描述，目前为止(此处存疑，我没探究具体是哪个内核版本)就定义了一个，即 `close-on-exec` 标志。
    - 对打开文件句柄的引用。

- 系统级的打开文件表(open file description table)。表中各个条目成为打开文件句柄( open file handle )，存储了打开文件的全部信息：

    - 当前文件的偏移量
    - 打开文件时所使用的状态标志位 (open() 的flag参数)
    - 文件访问模式 (open() 时设置的只读，只写或者读写)
    - 与信号驱动 I/O 相关的设置
    - 对文件 i-node 对象的引用

- 文件系统的i-node表。慈湖忽略 i-node 在磁盘和内存中的表示差异。大概有如下信息：

    - 文件类型
    - 一个指针，指向该文件所持有的锁的列表
    - 文件的各种属性，包括文件大小以及与不同类型操作相关的时间戳

所以，有可能会出现如下情况：

- 相同进程内的不同文件描述符指向同一个打开文件句柄。 (可能通过 dup()等操作)
- 不同进程内的文件描述符指向同一个打开文件句柄。 (可能在 fork() 后出现这种情况)
- 不同进程内的文件描述符指向不同的打开文件句柄，但是这些句柄都指向 i-node 表中相同的条目。 (可能在不同进程内 open 了同一个文件)
