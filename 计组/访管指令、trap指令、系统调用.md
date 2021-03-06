## 什么是管态和目态？
系统内核=核心态=cpu管态
上层应用程序=用户态=cpu目态

CPU状态分为管态和目态，CPU的状态属于程序状态字PSW的一位。
管态又称特权状态、系统态或核心态。通常，操作系统在管态下运行，CPU在管态下可以执行指令系统的全集。 可以使用特权指令。
目态又称常态或用户态。机器处于目态时，程序只能执行非特权指令。用户程序只能在目态下运行。 

一般说来，在单用户，单任务的计算机中不具有也不需要特权指令，而在多用户，多任务的计算机系统中，特权指令却是不可缺少的。它主要用于系统资源的分配和管理，包括改变系统的工作方式，检测用户的访问权限，修改虚拟存储器管理的段表，页表和完成任务的创建和切换等。

## 为什么要设置两个cpu状态？
一句话：多用户多任务系统的要求。

## 访管指令
用户程序执行过程中，需要调用操作系统提供的不同功能的子程序，也就是系统调用（如读/写文件、分配主存等），其中有些还必须执行硬件的特权指令（如 I/O指令）才能完成任务，因此，需要在目态（用户程序执行，CPU不处理特权指令）下提供一个渠道，来使操作系统执行相关特权指令，这个渠道就是“访管指令”。

访管指令调用

编译程序翻译过程中，将需调用操作系统功能的逻辑要求转换成访管指令，并设置一些参数；

处理器执行到访管指令时，会产生一个中断事件（1）（将操作系统程序的PSW装入程序状态字寄存器），实现用户程序与系统调用程序之间的转换；

系统调用按规定参数执行结束后，再返回到用户程序（2）（将用户程序的PSW重新装入程序状态字寄存器）。


## trap指令、访管指令、系统调用的关系？
>在系统调用中，TRAP负责由用户模式转换为内核模式，将返回地址保存至串行中以备后用。

>访管指令中包含TRAP指令,也正是从TRAP指令转入内核的.首先由用户程序主动调用访管指令,访管指令在用户态预先把数据存入寄存器,然后通过一条TRAP指令跳转到内核的系统调用处理程序.调用结束后首先返回访管指令,再从访管指令返回用户程序.如果你要问为什么访管指令这么神奇的话,那是因为访管指令是用汇编语言写的.


[系统调用](https://juejin.im/post/5c696189e51d45175b4a8bf6)
[系统调用](http://gityuan.com/2016/05/21/syscall/)