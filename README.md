## 要点简记
#### 1.隐式推导

make可以自动推导文件及其文件依赖关系后面的命令,所以我们没有必要在每一个**.o**文件后面都写上类似的命令，因为make 会自动识别并且自动推导命令.

```
objects = main.o
main.o:main.cc

main:$(objects)
	c++ $(objects) -o main
# 伪目标，防止目录下有同名为clean的文件造成make clean执行失败
.PHONY:clean
clean:
	rm $(objects) main
```

####2.在规则中使用通配符

```objects = $(wildcard *.o)```表示将object变量定义为所有.o文件组成的集合。在定义变量的时候，不能使用通配符，除非使用wildcard函数.

####3.引用其他的makefile文件
可以通过include 命令引用其他的makefile文件，如下所示:
```
	include foo.make *.mk
```
上面表示引用当前目录下的foo.make文件以及所有后缀为mk的文件。

####4.make 的工作方式

1.　读入所有的makefile
2.　读入被include 的其他makefile
3.　初始化文件中的变量
4.　推导隐晦规则，分析所有规则
5.　为所有的目标文件创建依赖关系链
6.　根据依赖关系，确定那些目标需要重新生成
7.　执行生成命令

####5.文件搜寻
为了指明更多的文件搜索路径，可以定义vpath变量。vpath的定义方式有下面３种:

1. vpath<pattern> <directories> **为符合模式pattern的文件指定搜索路径为directories**
2. vpath<pattern> **清除符合模式为pattern的文件搜索路径**
3. vpath　**清除所有已经设置的文件搜索路径**

```
vpath %.h ../header #表示要求make在"../headers"目录下搜索所有以".h"结尾的文件
```

####6.静态模式
静态模式可以更加容易地定义多目标的规则，可以让我们的规则更有弹性。

例子：

```
objects = foo.o bar.o
all: $(objects)
$(objects): %.o:%.c
	$(CC) -c $(CFLAGS) $< -o $@
```
上面的代码和下述代码等价：

```
foo.o: foo.c
	$(CC) -c $(CFLAGS) foo.c -o foo.o
bar.o: bar.c
	$(CC) -c $(CFLAGS) bar.c -o bar.o
```

可以使用filter函数去除指定模式的文件，如下所示：

```
files = foo.elc bar.o
$(filter %.o,$(files)):%.o:%.c
	$(CC) -c $(CFLAGS) $< -O $@
```

