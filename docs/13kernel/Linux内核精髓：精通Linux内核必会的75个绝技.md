## Linux内核精髓：精通Linux内核必会的75个绝技

### 一、内核入门（1-6）

#### 1、如何获取Linux内核

+ 总结
  + 核心是知道了**upstream**这个是啥意思，*看人家招聘上有写，自己不清楚呢*
  + 相对重要的内容是`git`的用法，自己在别的地方见过和用过，这么就暂不做为重点

##### 1.1、内核的种类

##### 1.2、Linux树

+ Linux-next树
  + 在确认各自之间可以兼容之后再添加到Linus树内
+ stable树
  + 只针对过去发布的内核版本进行bug修改
+ 开发树
  + 各子系统分别在不同的源码树中进行开发。**经过linux-next中的测试后再植入Linus树**
  + **linus树、开发树等作为所有树的根源，也称为upstream**
+ 发布版内核（distribution kernel）

##### 1.3、如何获取上游内核

+ Linux内核的开发都是在最新版上游内核的基础上进行的。
+ 验证兼容性

```shell
$gpg--keyserver wwwkeys.pgp.net--recv-keys 0x517DoFoE
$gpg-verify＜签名文件＞＜下载的文件＞
```

+ 使用git

```shell
$git clone git：//git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux-2.6
```

##### 1.4、如何获取发布版内核

+ Fedora14

+ Ubuntu10.10

  + ```shell
    #apt-get install linux-source
    #安装deb包后，会在/usr/src 下生成tar文件
    ```

##### 1.5、小结

##### 1.6、参考文献

+ 内核文档Documentation/development-process/*

#### 2、如何编译Linux内核

##### 2.1、内核编译的过程

+ 1、获取源代码，如有需要则进行修改。
+ 2、设置。*设置啥？*
+ 3、编译。
+ 4、根据发布版生成相应的源码包。*取决于，使不使用源码包管理系统*
+ 5、安装内核映像和模块。

##### 2.2、需要的源码包

##### 2.3、编译、安装上游内核

+ **kconfig**
  + 控制台的设置工具，`make menuconfig`，**会生成.config文件，不能手动编辑**
  + **每个架构都准备了默认的.config文件，执行`make defconfg`**
  + 基于图形，`make xconfig`
+ 使用Make的`-j`选项，**指并发性**，比如`-j 4`
+ 编译完成后安装，分2个步，**安装时要有root权限**
  + 一，是模块的安装，`make modules_install`，**安装在/lib/modules下**
  + 二，是安装内核二进制映像文件，生成并安装boot初始化文件系统映像文件。`make install`

##### 2.4、生成内核包

2.5、在源码树外编译模块

##### 2.6、交叉编译内核

+ **交叉编译**是指针对与正在执行编译的平台不同的其他平台生成二进制数据。（win下生成ARM的）

  + 两个变量，**ARCH，CROSS_COMPILE**

+ ```shell
  $make ARCH=arm CROSS_COMPILE=armv5tel-linux-uImage
  $make ARCH=arm CROSS_COMPILE=armv5tel-linux-modules
  ```

+ 

2.7、小结

##### 2.8、参考文献

+ Documentation/kbuild/*（内核源文档）

### 二、资源管理（7-16）

### 三、文件系统（17-21）

### 四、网络（22-27）

### 五、虚拟化（28-39）

### 六、省电（40-51）

### 七、调试（52-63）

### 八、概要分析与追踪（64-75）

