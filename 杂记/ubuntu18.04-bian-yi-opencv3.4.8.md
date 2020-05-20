# ubuntu18.04编译opencv3.4.8

## 编译选项

在opencv源码包文件夹内 mkdir build cd build

cmake命令在cmakelists.txt文件所在路径（即源码包）下执行

* 配置安装位置

  -D CMAKE\_INSTALL\_PREFIX=/usr/local

* 配置编译contrib包选项

  ```text
  -DOPENCV_EXTRA_MODULES_PATH=<opencv_contrib>/modules
  ```

## 配置环境变量

### 程序运行时去哪搜索需要加载的动态库？

### /etc/ld.so.conf.d/目录下文件的作用

![s](https://s1.ax1x.com/2020/04/29/Jo8HLq.png)

```text
sudo vim /etc/ld.so.conf.d/opencv.conf
```

里面添加 /usr/local/lib

### ldconfig命令

[https://man.linuxde.net/ldconfig](https://man.linuxde.net/ldconfig)

> ldconfig通常在系统启动时运行，而当用户安装了一个新的动态链接库时，就需要手工运行这个命令。

### cmake使用opencv

#### cmake环境变量

* 设置输出目录

  ```text
  set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
  set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
  ```

* 设置头文件搜索路径

  ```text
  INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/include)
  ```

* 设置非标准的库文件搜索路径

  \`\`\`

\`\`\` [https://zhuanlan.zhihu.com/p/50829542](https://zhuanlan.zhihu.com/p/50829542)

#### ..的含义是执行命令的当前目录，而不是文件所在目录

![ahh](https://s1.ax1x.com/2020/04/30/Jb9VQe.png) flower.jpg不是在项目路径下的resource路径下。 而是在当前命令行路径的上级路劲去找resource文件夹

## 参考

[https://www.cnblogs.com/fx-blog/p/8213704.html](https://www.cnblogs.com/fx-blog/p/8213704.html)

