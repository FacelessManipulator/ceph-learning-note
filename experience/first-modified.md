### 记录了我第一次尝试修改ceph源码的过程

在阅读源码的时候，对于一些DENC函数和一大堆宏操作不了解，想通过输入输出查看一下这堆函数的功能。为了查看函数的输入输出，其中一个方法就是将函数输入输出放入自己写的dout，重新编译后去查看日志获取值，步骤如下：

* 进入目标函数中，在函数头以及函数尾加入dout，为了查看日志方便，也可以自己编写一个mdout，将流重定向至自定义文件中方便debug
* 进入CEPHPATH/build 运行make -j4，等待编译完成

* 在build目录下运行:

```
# at path: CEPH_PATH/build
../src/stop.sh
MON=1 OSD=1 ../src/vstart.sh -d -n -l -x
cp ceph.conf keyring /etc/ceph/
# 如果使用了dout，则可以降低相应debug log等级至适当级别以减少debug日志量
ceph tell osd.0 injectargs --debug_osd 10/10
# 查看当前debug等级
ceph daemon osd.0 config show | grep debug_osd

# 执行相应操作脚本
./do_op.py
#查看日志
cat out/osd.0.log | grep <some unique features> | less
```



