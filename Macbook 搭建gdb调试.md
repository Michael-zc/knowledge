# Macbook 搭建gdb调试

<!--当前使用的操作系统版本为 macos High Sierra 10.13.3，对于不通的macos版本可能有区别-->



- #### 下载gdb

```
brew install https://raw.githubusercontent.com/Homebrew/homebrew-core/9ec9fb27a33698fc7636afce5c1c16787e9ce3f3/Formula/gdb.rb
```

安装 gdb8.0.1, 本人测试最新版gdb 8.1， 后续运行时会报错。

[[gdb doesn't work on macos High Sierra 10.13.3](https://stackoverflow.com/questions/49001329/gdb-doesnt-work-on-macos-high-sierra-10-13-3)](https://stackoverflow.com/questions/49001329/gdb-doesnt-work-on-macos-high-sierra-10-13-3)

安装完成后执行命令：

`echo "set startup-with-shell off" >> ~/.gdbinit`

- #### 生成证书

这是由于Mac os的安全机制阻止了我们的gdb对要调试的程序进行完全控制，对此我们要对gdb赋予合适的权限，首先我们要在keychain access里面添加相应的keychain (钥匙串)

![image-20180505091601589](/Users/luozhengcheng/Work/knowledge/image-20180505091601589.png)



##### 1.创建证书

![image-20180505091756728](/Users/luozhengcheng/Work/knowledge/image-20180505091756728.png)

填写如下信息：

![image-20180505092409250](/Users/luozhengcheng/Work/knowledge/image-20180505092409250.png)





##### 3. 指定有限期（本人测试有效天数最大可以设置为7300）

![image-20180505092609649](/Users/luozhengcheng/Work/knowledge/image-20180505092609649.png)

##### 4. 其他的都选择默认，最后创建完成

![image-20180505093936071](/Users/luozhengcheng/Work/knowledge/image-20180505093936071.png)

##### 5. 将证书从登录（login）钥匙串移动到系统钥匙串

- [ ] 打开系统钥匙串左上角的锁
- [ ] 在登录钥匙串中找到创建的gdb-cert证书。
- [ ] 把gdb-cert证书拖到系统钥匙串中。

##### 6. 选择"always trust"我们刚生成的证书

在系统钥匙串里找到gdb-cert证书，双击，并展开信任。

![image-20180505103730914](/Users/luozhengcheng/Work/knowledge/image-20180505103730914.png)



- #### 对gdb进行证书签名

完成上述步骤以后就可以退出keychain access了，但仅仅这样还是不够的，要对gdb进行签名，我们还需要杀死一个特殊的进程。

`sudo killall taskgated`

接下来可以对gdb进行签名了：

`codesign -fs gdb-cert /usr/local/bin/gdb`

此命令正常执行无任何输出。

<!--如果某天想取消对gdb的证书签名，可以使用‘codesign --remove-signature /usr/local/bin/gdb’来完成，前提是你的gdb-cert签名还在哦。-->



- ### 解除Mac对调试的保护

完成上述步骤后尝试用gdb调试代码。

--- CHECK IF IT WORKS ---

在我的环境中出现如下错误：

```
Starting program: /Users/luozhengcheng/Work/study/test
[New Thread 0x2803 of process 5805]
[New Thread 0x1803 of process 5805]
During startup program terminated with signal SIGTRAP, Trace/breakpoint trap.
```

则需要关闭macos的 System Integrity Protection，使macos运行debugging

1. 重启Mac 进入recovery mode （按住 `command+R ` 直到出现苹果logo）
2. 打开控制台
3. 输入命令 `csrutil enable --without debug`
4. 重启Mac



Debugging with gdb should now work as expected.

 