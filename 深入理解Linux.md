# 深入理解Linux

1. ## VFS

   - `VFS`是kernel中的软件层，存在于command和各类file system之间，用来兼容不同类型的file system
   - `cp /floppy/TEST /tmp/test` 比如floppy是MS-DOS的挂载点，tmp挂载的linux通用文件系统，cp和VFS交互，VFS负责调用两个文件系统的函数
   - VFS维护一个`common file model`来兼容不同类型的file system
     - super block
     - inode
     - dentry
     - file
     - file-->dentry-->inode-->block
     - superblock维护metadata，空闲inode，空闲block信息
   - file system的类型：
     - DIsk based：常用file system都是disk based
     - network：NFS，在不同主机间互传文件，同步文件
     - special：比如 `/proc` 是用来管理进程的

2. ## Special File System

   - 常见的special file system

     - 其中挂载点为none的文件系统不是用于用户交互的，而是便于kernel去执行一些操作
     - special filesystem不用和物理block device绑定，但是每个挂载的special file system会被分配一个虚拟的block device，0作为major number，minor number可为任意值

     | Name        | Mount Point   | 描述                                      |
     | ----------- | ------------- | ----------------------------------------- |
     | bdev        | none          | block devices                             |
     | eventpollfs | none          | 高效event polling 机制                    |
     | pipefs      | none          | 管道                                      |
     | proc        | /proc         | 通用access point to kernel data structure |
     | shm         | none          | 进程间通讯共享内存                        |
     | mqueue      | none          | POSIX message queue                       |
     | sockfs      | none          | sockets                                   |
     | sysfs       | /sys          | 通用access point to system data           |
     | tmpfs       | any(/tmp)     | 临时文件                                  |
     | usbfs       | /proc/bus/usb | USB                                       |

     

3. ## IPC(进程间通讯)

- 通过临时文件
  - 借助file system，very costly
  - 使用VFS的file lock来提供保护和同步
- Pipes and FIFOs
  - 最适宜用来做producer consumer的结构
- Semaphores
  - System V IPC
- Messages
  - 借助message queue：System V IPC messages；POSIX messages
  - message：short blocks of data
- shared memory regions
  - 最高效的方式
- Sockets
  - 在网络上互传消息



### 	3.1 Pipe

- Pipe的数据流传输是单向的

- Pipe可以看作一个打开的文件，但是没有任何被已挂载的file system管理的镜像

- 打开一个Pipe，会返回一对file descriptor，一个有读权限，一个有写权限

- 例子：

  ```shell
  ls | more
  
  // 上面这个command做的事情如下
  // 1. 主进程打开一个Pipe，返回一对file descriptor3和4
  // 2. 主进程fork一个子进程1
  // 3. 子进程1调用dup2(4,1)把descriptor4的copy到descriptor1(原标准输出)
  // 4. 子进程1关闭file descriptor3和4
  // 5. 子进程1执行ls，写descriptor1
  // 6. 主进程fork一个子进程2
  // 7. 子进程2调用dup2(3,0)把descriptor3的copy到descriptor0(原标准输入)
  // 8. 子进程关闭file descriptor3和4
  // 9. 子进程读file descriptor0执行more
  ```

  在c语言中，上述这些dirty work可以用popen()和pclose()来做

  ```
  // popen接受两个参数：1. 可执行文件pathname 2.数据流向
  // popen创建pipe，fork子进程1执行传入的pathname对应file
  // 如果数据流向是读，copy标准输入
  // 如果数据流向是写，copy标准输出
  ```

  

