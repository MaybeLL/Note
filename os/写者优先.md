# 写者优先

-[写者优先](xie-zhe-you-xian.md#写者优先)

-[实现一](xie-zhe-you-xian.md#实现一读写读写者直接和多个读者竞争)

-[实现二（精准唤醒写进程\)](xie-zhe-you-xian.md#实现二精准唤醒写进程)

-[为什么需要outmutex](xie-zhe-you-xian.md#为什么需要outmutex)

-[随机唤醒](xie-zhe-you-xian.md#随机唤醒)

-[如何实现A类进程对B类进程的优先级？](xie-zhe-you-xian.md#如何实现a类进程对b类进程的优先级)

-[经验](xie-zhe-you-xian.md#经验)

-[参考](xie-zhe-you-xian.md#参考)

## 写者优先

### 实现一（读－写－读：写者直接和多个读者竞争）

```cpp
semaphore ReaderWriterMutex = 1;    //实现读写互斥
int Rcount = 0;                //读者数量
semaphore CountMutex = 1;    //读者修改计数互斥
semaphore WriterMutex = 1;  //用于实现写者优先

writer(){
    while(true){
        P(WriterMutex); //无写进程时进入
        P(ReaderWriterMutex); //互斥访问共享文件
        write;
        V(ReaderWriterMutex);  //释放共享文件
        V(WriterMutex); //恢复对共享文件的访问
    }

}

reader(){
    while(true){
        P(WriterMutex); //无写进程时进入
        P(CountMutex);
        if(Rcount == 0)        //当第一个读者进来时，阻塞写者
            P(ReaderWriterMutex);
        ++Rcount;
        V(CountMutex);
        V(WriterMutex);//恢复对共享文件的访问

        read;

        P(CountMutex);
        --Rcount;
        if(Rcount == 0)
            V(ReaderWriterMutex);    //当最后一个读者离开后，释放写者
        V(CountMutex);
    }
}
```

存在的问题：读写进程都在WriterMutex处竞争，只有一个进程能WriterMutex。 写进程饥饿，且没有优先于读进程。 **想象这种场景**：一个写进程获得了WriterMutex正在写文件，此时有10万个读者和１个写者A在等它释放。

* 问题１:在写者Ａ之后的读进程也参与竞争，有可能比写进程先被唤醒。写者没有任何优先可言。（使用写者计数器＋读者请求阻塞信号量解决）
* 问题２：写进程释放WriterMutex，随机唤醒等待进程，写进程唤醒概率太低，发生饥饿。（使用outmutex解决）

结论：要避免写者和多个读者竞争WriterMutex，来达到优先唤醒写者的目的。

### 实现二（精准唤醒写进程）

```cpp
Reader()  
{  
    while(1)  
    {  
        wait(outmutex)//保证至多一个读进程请求R  
        wait(R)  

        wait(Rcount)  
        if(0 == readcount)  
            wait(fmutex)  
        readcount ++  
        signal(Rcount)  

        signal(R)  //此处释放Ｒ，一定是写进程抢到R，因为只有一个读和一个写请求R（第一个到来的写请求R）
        signal(outmutex)  

        ........  
        perform reading opreation  
        ........  

        wait(Rcount)  
        if(readcount == 0)  
            signal(fmutex)  
        signal(Rcount)  
    }  
}  

writer()  
{  
    while(1)  
    {  
        wait(Wcount)  
        if(writecount == 0)  
            wait(R)  
        writecount ++  
        signal(Wcount)  

        wait(fmutex)  
        ........  
        perform writing operation  
        ........  
        signal(fmutex)  

        wait(Wcount)  
        readcount--  
        if(readcount == 0)  
            signal(R)  
        signal(Wcount)  

    }  
}
```

### 为什么需要outmutex

outmutex作用：使得请求R的读进程最多１个。实现新来的写进程对已经等待的读者（成千上万）插队。 而读进程进入文件前都要先获得outmutex再获得R，新到来的写进程也要获得R。 获得R的读进程释放R时，只有新来的写进程在请求R，因为其他的读进程还在请求outmutex 考虑场景：使用类似哈希链表法的结构思考。 场景：读进程正在使用资源，还有一些读进程请求，但是进入的方式是先申请R再释放R才能进入，这没什么问题。但是如果这时新来一个写请求，按照写者优先，下一个进入文件的必须是这个写进程。但事实是新来的写进程要和等待堆积已久的千万读请求竞争R，无法实现新来的写进程对等待中的读进程的插队。

> 注：（多个读）－空（多个读者间不互斥，fmutex为空）

### 随机唤醒

进程退出临界区，释放信号量，此时唤醒进程是随机的。阻塞进程在池里而不是队列里。

### 如何实现A类进程对B类进程的优先级？

```cpp
A(){
    wait(count_mutex);
    if(A_count==0)
        wait(B);
    A_count++;
    signal(count_mutex);

    wait(fmutex);
    do something;
    signal(fmutex);

       wait(count_mutex);
    A_count--;
    if(A_count==0)
        signal(B);
    signal(count_mutex);
}

B(){
    wait(B);
    wait(fmutex);
    do something;
    signal(fmutex);
    signal(B);

}
```

### 经验

设计时从阻塞池抽象思考，分析代码时进程阻塞要具体到哪一行代码。 其次要注意写进程间没有同步关系，并不要求先发生的写请求就要先获得响应。读进程间也没有同步关系。 读写进程间存在同步关系（优先关系），有读写请求时，先读再写。通过一个信号量R实现。

读者写者问题实质是两类进程之间同步，同类进程间互斥。

* 两类进程间同步：计数器＋同步信号量（计数为１就请求信号量）
* 同类进程间互斥：互斥信号量

### 进程计数器实现＝计数变量＋访问互斥量

### 参考

[写者优先](https://love.junzimu.com/archives/2800)

