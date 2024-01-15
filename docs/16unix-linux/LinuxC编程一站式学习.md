### chap19、链接详解（269/478）

#### 19.1、多目标文件的链接

+ `gcc main.c stack.c -o main`
+ 分步编译`gcc -c main.c  gcc -c stack.c gcc main.o stack.o -o main`
+ **nm命令**查看目标文件的符号表
+ **`readelf -a main`**，可以看到main中的.bss段合并的另两个.o文件的bss段。
+ Q：两个.o文件，调换顺序有没有啥影响？
+ `ld --verbose`
+ 《GNU Binutils Documentation》

#### 19.2、定义和声明

+ 19.2.1、extern和static关键字
  + `gcc -Wall`的选项。
  + **为什么编译器在处理函数调用时需要知道函数原型？**

### 最后

#### 履历

+ 2021-04-15觉得这本书不错，看了一下第19章