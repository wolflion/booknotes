## Makefile_cmake_automake输出

+ 在项目构建中，小型的话，可能就自己手写Makefile，但如果大型的话，可以用以下2种方式生成，**但他们背后都是有依赖项的**
  + `cmake .` 依赖CMakeLists.txt，**需要手写CMakeLists.txt**
  + `./configure`  依赖 Makefile.in，**需要手写Makefile.am**

### 一、Makefile

+ ref
  + [makefile教程](https://www.ruanyifeng.com/blog/2015/02/make.html)，*阮一峰*
  + [GNU make](https://www.gnu.org/software/make/manual/make.html)，官网，*空闲了，要拜读一下*

#### 1、简单Makefile例子

```makefile
TARGET = hello
SOURCES = hello.c
DEPS =

CC = gcc
CFLAGS = -Wall -Werror
all:$(TARGET)
$(TARGET): $(SOURCES) $(DEPS)   #虽然写得有点抽象，主要是把变量用起来了，但能看懂
	$(CC) $(CFLAGS) -o $(TARGET) $(SOURCES)   #gcc -o hello hello.c，hello.c可以放最后吗？

clean:
	rm -f $(TARGET)
```

#### 2、makefile语法

+ makefile是由每条**规则（rule）**组成，rule由以下目录组成
  + 目标target：要生成的文件（*可以是执行文件，也可以是.o*），**也可以是伪目标**(*某个操作的名字*)，可以用`.phony`内置目标名
    + 简单例子中，目标就是`hello`，是个可执行文件
    + 简单例子中，`clean`，就是伪目标
  + 依赖项dependency：目标所依赖的文件
    + 简单例子中，目标不依赖啥文件，直接空
  + 命令command：生成目标所执行的操作
    + 简单例子中，`gcc -o hello hello.c`，*这是为了生成目标hello*
+ 规则中的有一个叫**前置条件**（prerequisites）来判断是否要构建目标
  + 冒号前面的叫**目标**，后面的叫**前置条件**（**通过判断 前置条件 是否修改【删除一个或者更改一个】，目标就要重新构建**
  + 目标是必须的
  + 前置条件 或 命令，至少要有一个存在，不能同时不存在
+ 为了让makefile更灵活性，可以使用**变量**和**函数**，还可以包含**判断和循环**
  + 变量（Variables）
    + 自定义变量：比如`SOURCES`，*自己定义了，才能用*
    + 内置变量（Variables Used by Implicit Rules）：比如`$(CC)`，*没有定义，就直接用了*
    + 自动变量（Automatic Variables）：`$@`
      + *暂时没用到，lionel，先不看*，也是从阮一峰blog看到的
  + 函数（Functions for Transforming Text）
    + 调用语法：`$(function arguments)`或`${function arguments}`
    + *可以学一下内置的*
    + *lionel，因为是没办法用自定义的*
  + 判断和循环，**使用的是Bash语法**  (*bash语法，我不太熟，lionel*)
    + 判断，`ifeq else endif`
    + 循环，`for i in $XX : do`

#### 3、make命令

+ `type make`看了一下，没写bultin-in，就不是内置的
+ `man make`也不行，`make -h`可以，一般man不行的时候，可以试一下`info make`，或者`make --help`
  + **man、info、help**3大命令
+ **make执行的时候，会去找Makefile**，找不到，会报错

### 二、Cmake&CMakeLists.txt 

+ ref
  + [cmake快速入门](https://durant35.github.io/2016/04/21/tool_CMake_%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/)  ，*找了一个例子*
  + [cmake实践](https://durant35.github.io/uploads/CMake%20Practice.pdf)，*也是个例子，未看*
  + [cmake](https://cmake.org/cmake/help/latest/)，官网

#### 1、cmake和make命令

+ `cmake`会生成Makefile，然后再执行`make`命令，这是步骤
  + `cmake .`生成Makefile时，**会去找CMakeLists.txt**

#### 2、简单CMakeLists.txt例子

+ **第一行，第二行的内容是固定的**

```txt
cmake_minimum_required(VERSION 3.0)  #最小版本，必须在第一行

project(hello)  #project是关键字，表示生成hello的项目，必须在第二行

add_executable(hello hello.c) #add_executable也是关键字，这里的hello是可执行文件，通过hello.c生成的
```

### 3、CMakeLists.txt的语法

+ *这部分要对应官网看下，以及上面ref里的2个例子*

### 三、automake

+ *还是要找实际的例子跑一下，现在用的例子，都是AI生成的*

+ 区别与联系
  + cmake是跨平台的
  + automake是unix上的
+ ref
  + [利用autotool生成Makefile](https://zhuanlan.zhihu.com/p/487937400) ，zhihu上的
  + [automake](https://www.gnu.org/software/automake/manual/automake.pdf) ，官网

#### 1、`./configure`命令

+ 下载一些开源软件，一般是让`./configure`一下，这是为了**生成Makefile**，然后再用`make`
+ **使用`./configure`时要确保安装了Autoconf**，*如何check是否安装了呢？lionel*
+ `./configure` 会**根据系统的配置信息来修改 `Makefile.in`**，再生成一个完整的 Makefile

#### 2、configure脚本是如何生成的

+ 2.1、用`autoscan` 生成 configure.scan，并把它修改为configure.ac
+ 2.2、 `aclocal` 命令。这将扫描 `configure.ac` 文件，并根据其中的宏定义生成 `aclocal.m4` 文件，**宏定义和宏展开**
+ 2.3、运行 `autoconf` 命令，会解析 `configure.ac` 文件，执行其中的宏和脚本，然后生成一个名为 `configure` 的脚本文件，**这里需要configure.ac和aclocal.m4两个文件**才行
+ 2.4、运行 `autoreconf` 命令（可选）

#### 3、Automake是如何生成Makefile.in的

+ Automake 使用这些宏来生成 Makefile.in。
+ *跟我看到的不一样嘛，lionel，自己要实践下*（2个AI回答得不一样嘛）

#### 4、Makefile.in和Makefile.am的区别

+ Makefile.am 是 Automake 生成的文件，它包含了构建项目的规则，但它并不是一个完整的 Makefile。
+ Makefile.in 是根据 Makefile.am 生成的文件，它是一个完整的 Makefile。**多一个配置信息**

#### 5、Makefile.in示例

+ 这是一个 Makefile.in 文件，通常用作模板文件，需要通过 `configure` 脚本生成最终的 Makefile 文件。在生成 Makefile 文件时，`@VARIABLE@` 类似的占位符会被实际的配置值替换。例如，`@CC@` 可能会被替换为实际的编译器名称。

```txt
# 编译器设置
CC = gcc
CFLAGS = -Wall -Wextra

# 目标文件
TARGET = myprogram

# 源代码文件
SRCS = main.c utils.c

# 生成目标文件
OBJS = $(SRCS:.c=.o)

# 默认目标
all: $(TARGET)

# 目标文件生成规则
$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) $(OBJS) -o $(TARGET)

# 依赖关系
main.o: main.c utils.h
utils.o: utils.c utils.h

# 清理目标文件和可执行文件
clean:
	rm -f $(OBJS) $(TARGET)
```

#### 5、Makefile.am示例

+ *我的疑问在于，makefile.am需要人工编写吗？* 【**需要手动编写**】

```txt
# 设置要生成的可执行文件名称
bin_PROGRAMS = myprogram

# 定义要编译的源代码文件
myprogram_SOURCES = main.c utils.c

# 定义要链接的库文件
myprogram_LDADD = -lm

# 清理规则
clean:
    rm -f $(bin_PROGRAMS)
```



### 其它构建工具

#### [meson](https://mesonbuild.com/)

### 最后

#### 感悟

+ 1、AI生成的吧，有点不对，但内容还是比我强太多了，*我只是整理其逻辑性，发现不对的地方*
+ 2、有些人会用，**也许，不是说特意去学它，而是项目中见多了，然后搜一下，多重反复**，（这是不是跟搞古董的**马未都**一样，他们也许没说真正去学过瓷器或其它啥的，就是见多了）

#### 履历

+ 0.1版，2024-01-05花了2个cubi整理了一下，中间跟AI碰撞了好几回，ubuntu环境不在，准备找个环境练个手后再更新