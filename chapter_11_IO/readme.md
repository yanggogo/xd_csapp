### UNIX I/O

1. 共享文件

子进程被fork出来后，会copy一份父进程的文件描述表（文件描述符表是每个进程都拥有独立的）

子进程的文件描述符表里的 fd 指针指向全局的 -- 文件表

父子进程共享该文件表里的数据，也就是seek偏移量。父子进程同一个文件的seek是一份。

fork 子进程其实只是增加了个指针执行 全局的文件表。

```python
import os
import time

# test1.txt 文件内容为 foobar

fd1 = open("test1.txt", mode='r', buffering=0)
# 如果这里的buffering不明确设置为0，也就是不适用buffer，那么默认buffer就是4096，
# fd1.read(3) 会一次读取所有 foobar到缓存，此时seek就是 end， 父进程的fd1.read(3) 就read出来是空的

# 可以尝试将buffering设置为4，子进程先读foob到缓存，但是只返回foo，此时seek为4，父进程再去读的时候，只能读到ar 
# buffering = 0 不适用缓存 son=foo father=bar
# buffering = 1  son=foo father=None —————— 很奇怪对吧，其实buffering=1 表示按行缓冲，不是buffer为1 字节大小的意思
# buffering = 2  son=foo, father=bar
# buffering = 4  son=foo, father=ar


pid = os.fork()

if pid == 0:
    #print("This is son. pid=%s" % os.getpid())
    son_read = fd1.read(3)
    # fd1.seek(0)
    print("son read %s" % son_read)
    #print("son read more %s" % fd1.read(3))
    time.sleep(13)
    print("son going exit")
    exit(0)
else:
    #print("This is father, father pid=%s. son pid=%s" % (os.getpid(), pid))
    time.sleep(3)
    father_read = fd1.read(3)
    print("father read %s" % father_read)
    exit(0)

