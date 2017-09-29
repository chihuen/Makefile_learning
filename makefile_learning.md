Makefile文件名

------

默认的情况下，make命令会在当前目录下按顺序找寻文件名为“GNUmakefile”、“makefile”、“Makefile”的文件，找到了解释这个文件。在这三个文件名中，最好使用“Makefile”这个文件名，因为，这个文件名第一个字符为大写，这样有一种显目的感觉。最好不要用 “GNUmakefile”，这个文件是GNU的make识别的

指定的话，你可以使用make的“-f”和“--file”参数，如：make -f Make.Linux或make --file Make.AIX



简单的例子如下

```
edit : main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o       /*注释:如果后面这些.o文件比edit可执行文件新,那么才会去执行下面这句命令*/
	cc -o edit main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o

main.o : main.c defs.h
	cc -c main.c
kbd.o : kbd.c defs.h command.h
	cc -c kbd.c
command.o : command.c defs.h command.h
	cc -c command.c
display.o : display.c defs.h buffer.h
	cc -c display.c
insert.o : insert.c defs.h buffer.h
	cc -c insert.c
search.o : search.c defs.h buffer.h
	cc -c search.c
files.o : files.c defs.h buffer.h command.h
	cc -c files.c
utils.o : utils.c defs.h
	cc -c utils.c
clean :
	rm edit main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o
```



Make工作过程

------

在默认的方式下，也就是我们只输入make命令。那么，

1. make会在当前目录下找名字叫“Makefile”或“makefile”的文件。
2. 如果找到，它会找文件中的第一个目标文件（target），在上面的例子中，他会找到“edit”这个文件，并把这个文件作为最终的目标文件。
3. 如果edit文件不存在，或是edit所依赖的后面的 .o 文件的文件修改时间要比edit这个文件新，那么，他就会执行后面所定义的命令来生成edit这个文件。
4. 如果edit所依赖的.o文件也不存在，那么make会在当前文件中找目标为.o文件的依赖性，如果找到则再根据那一个规则生成.o文件。（这有点像一个堆栈的过程）
5. 当然，你的C文件和H文件是存在的啦，于是make会生成 .o 文件，然后再用 .o 文件生成make的终极任务，也就是执行文件edit了。



使用变量

------

变量在声明时需要给予初值，而在使用时，需要给在变量名前加上“$”符号，但最好用小括号“（）”或是大括号“{}”把变量给包括起来。如果你要使用真实的“$”字符，那么你需要用“$$”来表示

```
objects = program.o foo.o utils.o
program : $(objects)
        cc -o program $(objects)

$(objects) : defs.h
```



```
x := foo
y := $(x) bar
x := later
```

其等价于：

```
y := foo bar
x := later
```

这种方法，前面的变量不能使用后面的变量，只能使用前面已定义好了的变量











伪目标

------

```
.PHONY : clean
clean :
	-rm edit $(objects)
```

如上面.PHONY 表明了clean是一个伪目标，伪目标不需要生成一个文件，同时，只要有这个声明，不管是否有“clean”文件，这个目标就是“伪目标”(注：-rm前面的-表明出错时不处理，继续执行后面的语句)

伪目标以生成目标作为依赖，如下伪目标all的依赖是prog1 prog2 prog3三个生成文件，直接通过一个make all生成3个目标文件：

```
all : prog1 prog2 prog3
.PHONY : all

prog1 : prog1.o utils.o
	cc -o prog1 prog1.o utils.o

prog2 : prog2.o
	cc -o prog2 prog2.o

prog3 : prog3.o sort.o utils.o
	cc -o prog3 prog3.o sort.o utils.o
```

同样，伪目标作为伪目标的依赖，如下，伪目标cleanall的依赖是cleanobj cleandiff两个伪目标，执行一个cleanall就相当于执行了make cleanobj 和make cleandiff两个：

```
.PHONY : cleanall cleanobj cleandiff

cleanall : cleanobj cleandiff
	rm program

cleanobj :
	rm *.o

cleandiff :
	rm *.diff
```





通配符

------

```
clean:
	rm -f *.o
```

*.o表明所有后缀为.o的文件



文件搜寻

------

Makefile文件中的特殊变量“VPATH”就是完成这个功能的，如果没有指明这个变量，make只会在当前的目录中去找寻依赖文件和目标文件。如果定义了这个变量，那么，make就会在当前目录找不到的情况下，到所指定的目录中去找寻文件了。

```
VPATH = src:../headers
```

上面的的定义指定两个目录，“src”和“../headers”，make会按照这个顺序进行搜索。目录由“冒号”分隔。（当然，当前目录永远是最高优先搜索的地方）



屏幕显示打印

------

通常命令会自动打印出来

如何屏蔽某一条命令的打印：

用“@”字符在命令行前，那么，这个命令将不被make显示出来

make带参数

如果make执行时，带入make参数“-n”或“--just-print”，**那么其只是显示命令，但不会执行命令**，这个功能很有利于我们调试我们的Makefile，看看我们书写的命令是执行起来是什么样子的或是什么顺序的。

而make参数“-s”或“--silent”则是全面禁止命令的显示。



需要顺序执行的命令

------

如果你要让上一条命令的结果应用在下一条命令时，**你应该使用分号分隔这两条命令**。比如你的第一条命令是cd命令，你希望第二条命令得在cd之后的基础上运行，那么你就**不能把这两条命令写在两行上**，而应该把这两条命令写在一行上，用分号分隔，如

```
exec:
	cd /home/hchen
	pwd
```

这个cd和pwd毫无关系

```
exec:
	cd /home/hchen; pwd
```

这个pwd打印的就是/home/hchen



忽略错误

------

忽略命令的出错，我们可以在Makefile的命令行前加一个减号“-”（在Tab键之后），标记为不管命令出不出错都认为是成功的

make参数

给make加上“-i”或是“--ignore-errors”参数，那么，Makefile中所有命令都会忽略错误。而如果一个规则是以“.IGNORE”作为目标的，那么这个规则中的所有命令将会忽略错误。这些是不同级别的防止命令出错的方法，你可以根据你的不同喜欢设置。

还有一个要提一下的make的参数的是“-k”或是“--keep-going”，这个参数的意思是，如果某规则中的命令出错了，那么就终止该规则的执行，但继续执行其它规则。



嵌套执行make

------

我们有一个子目录叫subdir，这个目录下有个Makefile文件，来指明了这个目录下文件的编译规则。那么我们总控的Makefile可以这样书写：

```
subsystem:
        cd subdir && $(MAKE)
```

进入当前目录下的subdir文件夹，执行make，等价于

```
subsystem:
        $(MAKE) -C subdir
```



宏定义命令

------

```
define run-yacc
yacc $(firstword $^)
mv y.tab.c $@
endef
```

定义这种命令序列的语法以“define”开始，以“endef”结束，以上以run-yacc定义了两行命令

调用如下

```
foo.c : foo.y
        $(run-yacc)
```

在这个命令包的使用中，命令包“run-yacc”中的“$^”就是“foo.y”， “$@”就是“foo.c”