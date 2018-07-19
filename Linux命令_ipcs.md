### linux命令：ipcs
####  ipcs   查看共享内存
	参数：
	* -m  打印出使用共享内存进行进程间通信的信息
	* -q   打印出使用消息队列进行进程间通信的信息
	* -s   打印出使用信号进行进程间通信的信息
	* -a   是默认的输出信息，打印出当前系统中所有的进程间通信方式的信息
    * -t    输出信息的详细变化时间
    * -p    输出ipc方式的进程ID
    * -c    输出ipc方式的创建者/拥有者
    * -u    输出当前系统下ipc各种方式的状态信息(共享内存、消息队列、信号)

* ipcs -m 输出如下：

* ![ipcs -m输出](https://raw.githubusercontent.com/lengender/MarkdownPhotos/master/ipcs_m.png)
	*  第一列是共享内存的key
	*  第二列是共享内存的变化shmid
	*  第三列是创建的用户owner
	*  第四列是权限perms
	*  第五列为创建的大小bytes
	*  第六列为连接到共享内存的进程数nattach
	*  第七列是共享内存的状态status。其中显示“dest”表示共享内存段已经被删除，但是还有用户在使用它，当该段内存的mode字段设置为SHM_DEST时就会显示"dest"。 当用户调用shmctl的IPC_RMID时，内存先查看多少个进程与这个内存关联着，如果关联数为0，就会销毁这段共享内存，否者设置这段内存的mod的mode位为SHM_DEST，如果所有进程都不用则删除这段共享内存。

####  ipcrm 移除一个消息队列，或者共享内存，或者一个信号集。同时会将与ipc对象相关联的数据也一起移除。
	* ipcrm -M shmkey  移除用shmkey创建的共享内存段
	* ipcrm -m shmid    移除用shmid标识的共享内存段
	* ipcrm -Q msgkey  移除用msqkey创建的消息队列
	* ipcrm -q msqid  移除用msqid标识的消息队列
	* ipcrm -S semkey  移除用semkey创建的信号
	* ipcrm -s semid  移除用semid标识的信号

