# 读写公平

## 实现一

```cpp
int fmutex = 1;
int readcount = 0;
int count_mutex = 1;

writer(){

    wait(fmutex);
    写文件;
    signal(fmutex);

}

reader(){


    wait(count_mutex);
    if(readcount==0){
        wait(fmutex);
    }
    readcount++;
    signal(count_mutex);


    读文件；

    wait(count_mutex);
    readcount--;
    if(readcount==0){
        signal(fmutex);
    }
    signal(count_mutex);

}
```

存在问题：后来的读者不申请fmutex，直接进入。当前写者在写文件，会造成同时读写。

## 正确实现

```cpp
int wr = 1;
int fmutex = 1;
int readcount = 0;
int count_mutex = 1;

writer(){
    wait(wr);//
    wait(fmutex);
    写文件;
    signal(fmutex);
    signal(wr);//
}

reader(){
    wait(wr);//

    wait(count_mutex);
    if(readcount==0){
        wait(fmutex);
    }
    readcount++;
    signal(count_mutex);

    signal(wr);//

    读文件；

    wait(count_mutex);
    readcount--;
    if(readcount==0){
        signal(fmutex);
    }
    signal(count_mutex);

}
```

