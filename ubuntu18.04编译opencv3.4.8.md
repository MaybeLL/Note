## 编译选项
在opencv源码包文件夹内
mkdir build
cd build

cmake命令在cmakelists.txt文件所在路径（即源码包）下执行

* 配置安装位置
-D CMAKE_INSTALL_PREFIX=/usr/local
* 配置编译contrib包选项
```
-DOPENCV_EXTRA_MODULES_PATH=<opencv_contrib>/modules 
```
* 配置sift等专利算法
-D OPENCV_ENABLE_NONFREE:BOOL=ON

> 不然会在８７％处报错：error: ‘cv::xfeatures2d::sift’ has not been declared

## 配置环境变量
### 程序运行时去哪搜索需要加载的动态库？
### /etc/ld.so.conf.d/目录下文件的作用

![s](https://s1.ax1x.com/2020/04/29/Jo8HLq.png)

```
sudo vim /etc/ld.so.conf.d/opencv.conf 
```
里面添加
/usr/local/lib  