# 示例1
# main:main.o test1.o test2.o
# 	gcc main.o test1.o test2.o -o main
	
# main.o:main.c test.h
# 	gcc -c main.c -o main.o

# test1.o:test1.c test.h
# 	gcc -c test1.c -o test1.o

# test2.o:test2.c test.h
# 	gcc -c test2.c -o test2.o

# 示例2：wildcard函数在我们引用变量的时候，会帮我们展开
# OBJ=$(wildcard *.c)
# test: $(OBJ)
# # 输出test
# 	@echo $@
# # 输出test2.c main.c test1.c
# 	@echo $^ 
# # gcc -o $@ $^

# 示例3：引用变量
# OBJ = main.o test1.o
# test: $(OBJ)
# #	gcc -o test $(OBJ)
# #	gcc -o test $^
# 	@echo $^

# 示例4：%通配符，
# "%.o" 把我们需要的所有的 ".o" 文件组合成为一个列表，从列表中挨个取出的每一个文件
# "%" 表示取出来文件的文件名（不包含后缀），然后找到文件中和 "%"名称相同的 ".c" 文件，然后执行下面的命令，直到列表中的文件全部被取出来为止
test: test.o test1.o
	@echo "-------"
	@echo $@ 
	@echo $^
#	@touch test
%.o: %.c
	@echo "======="
	@echo $@ 
	@echo $^

# 示例5：VPATH和vpath变量
# 1. VPATH 是变量，更具体的说是环境变量，Makefile 中的一种特殊变量，使用时需要指定文件的路径
# 2. vpath 是关键字，按照模式搜索，也可以说成是选择搜索。搜索的时候不仅需要加上文件的路径，还需要加上相应限制的条件
# 	 vpath [PATTERN] [DIRECTORIES]
# 	 eg. vpath test.c src -> 在 src 路径下搜索文件 test.c
# VPTAH = src car
# test: test.o
# 	gcc -o $@ $^

# 示例5.1 - VPATH
# VPATH=src include
# main:main.o list1.o list2.o
#     gcc -o $@ $<
# main.o:main.c
#     gcc -o $@ $^
# list1.o:list1.c list1.h
#     gcc -o $@ $<
# list2.o:list2.c list2.h
#     gcc -o $@ $<

# 示例5.2 - vpath
# vpath %.c src
# vpath %.h include
# main:main.o list1.o list2.o
#     gcc -o $@ $<
# main.o:main.c
#     gcc -o $@ $^
# list1.o:list1.c list1.h
#     gcc -o $@ $<
# list2.o:list2.c list2.h
#     gcc -o $@ $<

# 伪目标，不是具体的文件。不会与第一个目标文件相关联，所以我们在执行 make 的时候也不会执行下面的命令
.PHONY: clean
clean:
	@echo "clean"
  # rm -rf *.o test