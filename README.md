# 默认执行的规则 

默认情况下，make 执行的是 Makefile 中的第一规则（Makefile 中出现的第一个依赖关系），此规则的第一目标称之为“最终目标”或者是“终极目标”

# 变量

变量可以出现在我们规则的目标中，也可以是我们规则的依赖中

## 变量的替换引用（常用）

表达式中使用了 "%" 这个字符，这个字符的含义就是自动匹配一个或多个字符。在开发的过程中，我们通常会使用这种方式来进行变量替换引用的操作

```Makefile
foo:=a.c b.c d.c
# 括号中的变量使用的是变量名而不是变量名的引用，变量名的后面要使用冒号和参数选项分开，表达式中间不能使用空格
obj:=$(foo:%.c=%.o)
All:
    @echo $(obj)
```

```Makefile
foo:=a123c a1234c a12345c
obj:=$(foo:a%c=x%y)
All:
    @echo $(obj)
```

## 变量的嵌套引用（少用）

变量的嵌套引用的具体含义是这样的，我们可以在一个变量的赋值中引用其他的变量，并且引用变量的数量和和次数是不限制的

```Makefile
foo=bar
bar=test
var:=$($(foo))
All:
    @echo $(var) # 结果为test
```

```Makefile
first_pass=hello
bar=first
foo=pass
var:=$(bar)_$(foo)
all:
    @echo $(var)
```

## 变量赋值

简单赋值 ( := ) 编程语言中常规理解的赋值方式，只对当前语句的变量有效。

```Makefile
x:=foo
y:=$(x)b
x:=new
test：
  @echo "y=>$(y)"
  @echo "x=>$(x)"

// 输出
y=>foob
x=>new
```

递归赋值 ( = ) 赋值语句可能影响多个变量，所有目标变量相关的其他变量都受影响。

```Makefile
x=foo
y=$(x)b
x=new
test：
  @echo "y=>$(y)"
  @echo "x=>$(x)"

// 输出
y=>newb
x=>new
```

条件赋值 ( ?= ) 如果变量未定义，则使用符号中的值定义变量。如果该变量已经赋值，则该赋值语句无效。
追加赋值 ( += ) 原变量用空格隔开的方式追加一个新值。

# 条件表达式 & 循环语句

在条件表达式中不能使用自动化变量（$@、$^、$<...），自动化变量在规则命令执行时才有效

```Makefile
ifeq ($(CC),gcc)
  ...
else
	...
endif
```

循环写法：
```Makefile
all_images: .minideb
	for dir in $(SUBDIRS); do \
		echo "build $$dir"; \
		$(MAKE) -C $$dir; \
	done
```

# 是否要生成目标文件

如果没有生成目标文件，那么重复执行make指令的时候，也会重复执行相关指令。如果生成了目标文件，而且依赖文件没有更新，那么生成目标文件的模块指令就不会执行

```Makefile
test: test.o test1.o
	@echo "-------"
	@echo $@ 
	@echo $^
#	@touch test
%.o: %.c
	@echo "======="
	@echo $@ 
	@echo $^
```

# 伪目标

通常一个伪目标不作为另外一个目标的依赖，这是因为当一个目标文件的依赖包含伪目标时，每一次在执行这个规则伪目标所定义的命令都会被执行（因为它作为规则的依赖，重建规则目标时需要首先重建规则的所有依赖文件）

```Makefile
# 就算Makefile目录下有debug文件，但是该debug目标规则的命令还是会被执行
debug: .coo
	@echo "debug"

.coo:
	@echo ".coo"
```

# 常用函数

- 字符串处理函数（patsubst、subst、strip、findstring、filter、sort、word）
- 文件名操作函数
	- 例如获取文件的路径，去除文件的路径，取出文件前缀或后缀等等）
	- 每个函数的参数字符串都会被当作或是一个系列的文件名来看待
	- dir、notdir、suffix、basename、addsuffix、addperfix、join、wildcard
- 其他函数
	- forearch、if、call（用来创建新的参数化的函数）、origin

格式：$(<function> <arguments>) 或者是 ${<function> <arguments>}

# shell解析

通常系统中可能存在不同的 shell 。但是 make 处理 Makefile 过程时，如果没有明确的指定，那么对所有规则中的命令行的解析使用bin/sh来完成

## 命令回显（常用于调试）

当使用 make 的时候机加上参数-n或者是--just-print ，执行时只显示所要执行的命令，但不会真正的执行这个命令。只有在这种情况下 make 才会打印出所有的 make 需要执行的命令，其中包括了使用的“@”字符开始的命令。这个选项对于我们调试 Makefile 非常的有用，使用这个选项就可以按执行顺序打印出 Makefile 中所需要执行的所有命令。而 make 参数-s或者是--slient则是禁止所有的执行命令的显示。就好像所有的命令行都使用“@”开始一样

```sh
make -n
make --just-print
```

## 命令的执行（注意多行命令的问题）

（重点）当规则中的目标需要被重建的时候，此规则所定义的命令将会被执行，如果是多行的命令，那么每一行命令将是在一个独立的子 shell 进程中被执行。因此，多命令行之间的执行命令时是相互独立的，相互之间不存在以来

在 Makefile 中书写在同一行中的多个命令属于一个完整的 shell 命令行，书写在独立行的一条命令是一个独立的 shell 命令行。因此：在一个规则的命令中命令行 “cd”改变目录不会对其后面的命令的执行产生影响。就是说之后的命令执行的工作目录不会是之前使用“cd”进入的那个目录。如果达到这个目的，就不能把“cd”和其后面的命令放在两行来书写。而应该把这两个命令放在一行上用分号隔开。这样才是一个完整的 shell 命令行

```Makefile
foo: bar/lose
	cd bar;gobble lose >../foo
```

```Makefile
# 和上述例子一样效果
foo:bar.lose
	cd bar; \
	gobble lose > ../foo
```

make 对所有规则的命令的解析使用环境变量“SHELL”所指定的那个程序。在 GNU make 中，默认的程序时 “/bin/sh”。不像其他的绝大多数变量，他们的只可以直接从同名的系统环境变量那里获得。make 的环境变量 “SHELL”没有使用环境变量的定义。因为系统环境变量“SHELL”指定的那个程序被用来作为用户和系统交互的接口程序，他对于不存在直接交互过程的 make 显然不合适。在 make 环境变量中“SHELL”会被重新赋值；他作为一个变量我们也可以在 Makefile 中明确的给它赋值，变量“SHELL“的默认值时“/bin/sh”

```Makefile
MODULE = $(shell env GO111MODULE=on $(GO) list -m)
```

## 并发执行命令（一般不使用）

GNU make 支持同时执行多条命令。通常情况下，同一时刻只有一个命令在执行，下一个命令只有在当前命令结束之后才能够开始执行。不过可以通过 make 命令行选项 "-j" 或者 "--jobs" 来告诉 make 在同一时刻可以允许多条命令同时执行

GNU make 支持同时执行多条命令。通常情况下，同一时刻只有一个命令在执行，下一个命令只有在当前命令结束之后才能够开始执行。不过可以通过 make 命令行选项 "-j" 或者 "--jobs" 来告诉 make 在同一时刻可以允许多条命令同时执行

问题：

- 多个同时执行的命令的输出信息将同时被输出到终端。当出现错误时很难根据一大堆凌乱的信息来区分那条命令执行错误
- 在同一时刻可能会存在多个命令执行的进程同时读取到标准输入，但是对于白哦准输入设备来说，在同一时刻只能存在一个进程访问它。就是说在某个时间点，make只能保证此刻正在执行的进程中的一个进程读取标准输入流。而其他的进程键的标准输入流将设置为无效。因此在此一时刻多个执行命令的进程中只有一个进程获得标准输入，而其他的需要读取标准输入流的进程由于输入流无效而导致致命的错误

# include的作用（大型项目）

当 make 读取到 "include" 关键字的时候，会暂停读取当前的 Makefile，而是去读 "include" 包含的文件，读取结束后再继读取当前的 Makefile 文件。"include" 使用的具体方式如下：

```Makefile
include <filenames>
```

include 通常使用在以下的场合：
- 在一个工程文件中，每一个模块都有一个独立的 Makefile 来描述它的重建规则。它们需要定义一组通用的变量定义或者是模式规则。通用的做法是将这些共同使用的变量或者模式规则定义在一个文件中，需要的时候用 "include" 包含这个文件。
- 当根据源文件自动产生依赖文件时，我们可以将自动产生的依赖关系保存在另一个文件中。然后在 Makefile 中包含这个文件。

# 嵌套执行make（大型项目）

每个模块可能都会有自己的编译顺序和规则

```Makefile
subsystem:
    cd subdir && $(MAKE) MAKEFLAGS=
```

```Makefile
# 等效上述例子
subsystem
    $(MAKE) -C subdir
```

## export的使用

使用 make 嵌套执行的时候，变量是否传递也是我们需要注意的。如果需要变量的传递，那么可以这样来使用，`<variable>` 是变量的名字，不需要使用 "$" 这个字符。如果所有的变量都需要传递，那么只需要使用 "export" 就可以，不需要添加变量的名字

```Makefile
export <variable> 
unexport <variable>
```

## 案例

总控 Makefile：

- 由于这些“工作目标目录”被设成 .PHONY 的依赖文件，所以即使工作目标已经更新，此规则仍旧会进行更新动作。使 --directory(-C) 选项的目的是要让 make 在读取 Makefile 之前先切换到相应的 "工作目录"

```Makefile
lib_codec := lib/codec
lib_db    := lib/db
lib_ui     := lib/ui
libraries   := $(lib_codec) $(lib_db) $(lib_ui)
player    := app/player

.PHONY : all $(player) $(libraries)
all : $(player)

# 一个规则在工作目标上列出了所有的子目录
$(player) $(libraries) :
    $(MAKE) -C $@

$(player) : $(libraries)
$(lib_ui) : $(lib_db) $(lib_codec)
```

# 目标类型

## 强制目标

如果一个目标中没有命令或者是依赖，并且它的目标不是一个存在的文件名，在执行此规则时，目标总会被认为是最新的。就是说：这个规则一旦被执行，make 就认为它的目标已经被更新过。这样的目标在作为一个规则的依赖时，因为依赖总被认为更新过，因此作为依赖在的规则中定义的命令总会被执行

```Makefile
# 使用 "FORCE" 目标的效果和将 "clean" 声明为伪目标的效果相同
clean:FORCE
    rm $(OBJECTS)
FORCE:
```

## 空目标（	常用）

空目标文件是伪目标的一个变种，此目标所在的规则执行的目的和伪目标相同——通过 make 命令行指定将其作为终极目标来执行此规则所定义的命令。和伪目标不同的是：这个目标可以是一个存在的文件，但文件的具体内容我们并不关心，通常此文件是一个空文件

空目标文件只是用来记录上一次执行的此规则的命令的时间。在这样的规则中，命令部分都会使用 "touch" 在完成所有的命令之后来更新目标文件的时间戳，记录此规则命令的最后执行时间。make 时通过命令行将此目标作为终极目标，当前目标下如果不存在这个文件，"touch" 会在第一次执行时创建一个的文件

通常，一个空目标文件应该存在一个或者多个依赖文件。将这个目标作为终极目标，在它所依赖的文件比它更新时，此目标所在的规则的命令行将被执行。就是说如果空目标文件的依赖文件被改变之后，空目标文件所在的规则中定义的命令会被执行

```Makefile
# 执行 "make print" ,当目标文件 "print" 的依赖文件被修改之后，命令 "lpr -p $?" 都会被执行，打印这个被修改的文件
# 如果不执行touch print，那么每次make print的话，该print规则的指令都会被执行
print: foot.c bar.c
    lpr -p $?
    touch print
```

## 特殊目标

- .PHONY，这个目标的所有依赖被作为伪目标。伪目标是这样一个目标：当使用 make 命令行指定此目标时，这个目标所在的规则定义的命令、无论目标文件是否存在都会被无条件执行

## 多规则目标（比较少用）

# 控制函数（两种）

Makefile 中提供了两个控制 make 运行方式的函数。其作用是当 make 执行过程中检测到某些错误时为用户提供消息，并且可以控制 make 执行过程是否继续

- error函数，产生致命错误，并提示 "TEXT..." 信息给用户，并退出 make 的执行
- warning函数，函数 "warning" 类似于函数 "error" ，区别在于它不会导致致命错误（make不退出），而只是提示 "TEXT..."，make 的执行过程继续

```Makefile
ERROR1=1234
all:
    ifdef ERROR1
    $(error error is $(ERROR1))
    endif  
```

```Makefile
ERR=$(error found an error!)
.PHONY:err
err:;$(ERR)
```


# 默认定义的变量

- $(MAKE)
- $(CC)
- $(CURDIR)，此变量代表 make 的工作目录。当使用 make 的选项 "-C" 的时候，命令就会进入指定的目录中，然后此变量就会被重新赋值
- $(SHELL)
- $(MAKEFLAGS)，包含了 make 的参数信息，如果执行总控 Makefile 时，make 命令带有参数或者在上层的 Makefile 中定义了这个变量，那么 MAKEFLAGS 变量的值将会是 make 命令传递的参数，并且会传递到下层的 Makefile 中，这是一个系统级别的环境变量
















