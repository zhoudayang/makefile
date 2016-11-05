## make 要点简记
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
	　-rm $(objects) main
```

####2.在规则中使用通配符

```objects = $(wildcard *.o)```表示将object变量定义为所有.o文件组成的集合。在定义变量的时候，不能使用通配符，否则上述变量objects的值就为***.o**,除非使用**wildcard**函数.

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
3. vpath **清除所有已经设置的文件搜索路径**

```
vpath %.h ../header #表示要求make在"../headers"目录下搜索所有以".h"结尾的文件
```

####6.静态模式
静态模式可以更加容易地定义多目标的规则，可以让我们的规则更有弹性。
语法：
```
<targets ...>:<target-pattern>:<prereq-patterns>
	<commands>
```
在上面的命令中：
1. targets定义了一系列的目标文件，是目标的一个集合，可以有通配符。
2. target-pattern指明了target的模式，也就是目标集模式。
3. prereq-patterns是目标的依赖模式，它是对target-pattern形成的模式再进行一次依赖目标的定义。

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
	$(CC) -c $(CFLAGS) $< -o $@
```

####7.嵌套执行make
在一些大型的工程中，我们会把不同的模块或者不同功能的源文件放在不同的目录中，我们可以在每个目录中都书写一个该目录下的makefile文件，这有利于维护我们的makefile.

例如我们有一个子目录叫subdir, 这个目录下有一个makefile文件，来指明了这个目录下的文件的编译规则，那么我们总控的makefile文件可以这么写。
```
subsystem:
	cd subdir $$(MAKE)
```

其等价于:
```
subsystem:
	$(MAKE) -C subdir
```

如果想要传递变量到下级makefile，可以使用这样的声明：
```
export <variable ...>
```

如果不想让某些变量传递到下级makefile中，可以这样声明:
```
unexport <variable ...>
```

#### 8.变量中的变量
在定义变量的值时，可以使用其它变量来构造变量的值。有下述两种方式：

###### 1. 采用=定义变量
使用=定义变量可以使用未定义的值，也可以是后面定义的值，如下所示：
```
foo = $(bar)
$bar=huh?
```

上面例子中$(foo)最终的值为huh?

###### 2.采用:=定义变量
使用:=定义变量，必须使用当前已经定义好的变量，不能使用当前未定义的变量。

##### 9.追加变量值
我们可以使用"+="操作符给变量追加值。例如:
```
objects: main.o foo.o bar.o utils.o
objects+=another.o
```
这样，上面例子中的$(objects)最终的值为```main.o foo.o bar.o utils.o another.o```

####10.使用条件判断
用两个例子来介绍这一部分的内容：
例１:
```
libs_for_gcc = -lgnu
normal_libs = 
foo:$(objects)
ifeq ($(CC),gcc)
$(CC) -o foo $(objects) $(libs_for_gcc)
else
$(CC) -o foo $(objects) $(normal_libs)
endif
```
**与ifeq相对应的有ifneq,即如果不等于**
例2：
```
bar = 
foo = $(bar)
ifdef foo
frobozz=yes
els
frobozz=no
endif
```
类似于C++中宏定义中的#ifdef

####11.函数
##### 1.字符串处理函数
###### 1.subst
```
$(subst <from>,<to>,<text>)
名称:字符串替换函数-subst函数
功能:将字符串<test>中的<from>字符串替换为<to>
返回：函数返回被替换过后的字符串　
```

###### 2.patsubst
```
$(patsubst <pattern>,<replacement>,<text>)
名称：模式字符串替换函数-patsubst
功能：查找<test>中的单词是否符合模式pattern,如果匹配，就以<replacement>进行替换，这里的<pattern>可以包括通配符"%"表示任意长度的字符串。
```
例子：
```
patsubst(%.c,%.o,x.c.c bar.c)
将字符串"x.c.c bar.c"中符合模式[%.c]的单词替换为[%.o],故返回的结果为x.c.o bar.o
```

######3.strip
```
$(strip <string>)
名称：去空格函数
功能：去掉<string>字符串中开头和结尾的空字符串
返回：返回去掉空格的字符串值
```

######４.findstring
$(findstring <find>,<in>)
名称：查找字符串函数-findstring
功能：在字符串<in>中查找<find>字符串
返回：如果找到，返回<find>,否则返回空字符串

######5.filter
```
$(filter <pattern...>,<text>)
名称：过滤函数
功能：以<pattern>模式过滤<text>字符串中的单词，保留符合模式<pattern> 的单词。可以有多个模式
```

例子：
```
sources = foo.c bar.c baz.s ugh.h
foo:$(source)
cc $(filter %.c %.s,$(sources)) -o foo
```
其中$(filter %.c %.s,$(sources))返回的值为"foo.c bar.c baz.s"

######6.filter-out
```
$(filter-out <pattern ...>,<text>)
名称：反过滤函数-filter-out
功能：以<pattern>模式过滤<text>字符串中的单词，去掉符合模式<pattern>的单词，可以有多个模式
返回：返回不符合模式<pattern>的字符串。
```

例子：
```
objects = main1.o main2.o foo.o bar.o
mains=main1.o main2.o
$(filter-out $(mains),$(objects))
上述经过filter-out的返回值为bar.o以及foo.o
```

######7.sort
```
$(sort <list>)
名称：排序函数-sort
功能：给字符串<list>中的单词排序
返回：返回排序后的字符串
示例:$(sort foo bar lose)返回"bar foo lose"
备注：sort函数会去除list之中重复的单词
```

######8.word
```
$(word <n>,<text>)
名称：去单词函数-word
功能：取出字符串text中的第<n>个单词（从１开始）
返回：返回字符串<text> 中的第<n>个单词，若<n>比　<text>中的单词数要大，那么返回空字符串
示例:$(word 2,foo bar baz)　返回值为bar
```

#####2.文件名操作函数
######1.dir
```
$(dir <names ...>)
名称：取目录函数　--dir
功能：从文件名序列<names>中取出目录部分，目录部分指的是最后一个反斜杠　"/"之前的部分，如果没有反斜杠，则返回“./”
返回：返回文件名序列<names>中的目录部分
示例：$(dir src/foo.c hacks)返回值为　"src/ ./"
```
######2.notdir
```
$(notdir <names...>)
名称：取文件函数
功能：从文件名序列<names>中取出非目录的部分。菲目录部分是最后一个／之后的部分
返回：返回文件名序列<names>中的非目录部分
示例:$(notdir src/foo.c hacks)返回值为"foo.c hacks"
```
######3.suffix
```
$(suffix <names...>)
名称：取后缀函数--suffix
功能：从文件名序列<names> 中取出各个文件名的后缀
返回：返回文件名序列<names>的后缀序列，如果文件没有后缀，则返回空字符串。
示例:$(suffix src/foo.c src-1.0/bar.c hacks) 返回值为".c .c"
```

######4.basename
```
名称：取前缀函数--basename
功能：从文件名序列中取出各个文件名的前缀部分
返回：返回文件名序列<names>的前缀序列，如果文件没有前缀，则返回空字符串。
示例：$(basename src/foo.c src-1.0/bar.c hacks)的返回值为"src/foo src-1.0/bar hacks"
```
######5.addsufix
```
$(addsuffix <suffix>,<names...>)
名称：加后缀函数-addsuffix
功能：把后缀<suffix>加到　<names>中的每个单词后面
返回：返回加过后缀的文件名序列
示例：$(addsuffix .c,foo bar)返回值为"foo.c bar.c"
```

######6.addprefix
```
$(addprefix <suffix>,<names...>)
名称：加后缀函数--addsuffix
功能：把后缀<suffix>加到<names>中的每个单词后面。
返回：返回加过后缀的文件名序列
示例：$(addprefix .c,foo bar)返回值为"foo.c bar.c"
```

######7.join
```
$(join <list1>,<list2>)
名称：连接函数--join
功能：把<list2>中的单词相应地添加到<list1>的单词后面。如果<list1>中的单词个数要比<list2>多，那么<list1>中多出来的单词将保持原样。如果<list2>中的单词数量要比<list1>多，那么<list2>中多出来的单词要被复制到<list1>中。　
返回：返回连接过后的字符串　
示例：$(join aaa bbb,111 222 333) 返回值为"aaa111,bbb222,3333"
```
##### 3.foreach函数
foreach函数用于循环。语法如下：
```
$(foreach <var>,<list>,<text>)
这个函数的意思是，将<list>中的单词逐一取出放到参数<var>所指定的变量之中，然后再执行<text>所包含的表达式。每次<text>会返回一个字符串，循环过程中,<text>返回的每个字符会以空格分隔，当循环结束的时候，<text> 返回的每个字符串所组成的整个字符串会是foreach函数的返回值。
```
例子：
```
names:=a b c d
files := $(foreach n,$(names),$(n).o)
执行完毕后，变量$(files)的值为"a.o b.o c.o d.o"
```

##### 4.if函数
```
1.$(if <condition>, <then-part>)
2.$(if <condition>,<then-part>,<else-part>)
```
```
if函数可以包含else或者不包含。即if函数的参数可以是两个，也可以是３个。<condition>参数是if的表达式，如果器返回为非空字符串，那么这个表达式就为真，于是<then-part>会被计算，否则<else-part>会被计算。

如果<condition>为真，那么<then-part>会为整个函数的返回值，否则<else-part>会作为函数ｕｄｅ返回值。如果<else-part>没有定义，那么函数会返回空字符串。
```