# main: main.cpp
# 	g++ -o main main.cpp

# 示例1：shell循环
# 代码中实现的部分是用当前目录下的 Makefile 控制其它子模块中的 Makefile 的运行
# 1. 子目录执行 make 出现错误时，make 不会退出
# 2. 没有用到 make 对目录的并行处理功能由于规则的命令是一条完整的 shell 命令，不能被并行处理
# SUBDIRS = foo bar baz
# subdirs:
# 	for dir in $(SUBDIRS); do echo $$dir;done
#	for dir in $(SUBDIRS); do $(MAKE) -C $$dir;done

# 示例2（改进）
# SUBDIRS=foo bar baz
# .PHONY: subdirs $(SUBDIRS)
# subdirs: $(SUBDIRS)
# $(SUBDIRS):
# 	$(MAKE) -C $@
# foo: baz # 这是一个没有命令行的规则，这个规则是用来规定三个子目录的编译顺序

# 示例3
# 约定使用 "all" 的伪目标来作为最终目标，它的依赖文件就是要生成的可执行文件。这样的话只需要一个 make 命令，就会同时生成三个可执行文件
# .PHONY: all
# all: test1 test2 test3
# test1: test1.o
# 	gcc -o $@ $^
# test2: test2.o
# 	gcc -o $@ $
# test3: test3.o
# 	gcc -o $@ $

# .PHONY: debug
# debug:
# 	@echo $(MAKE)

debug: .coo
	@echo "debug"

.coo:
	@echo ".coo"