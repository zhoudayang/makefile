1. 使用自动命令推导，*.o和*.cc或者*.c文件有自动命令推导，自动执行cc -c命令.
2. ```.PHONY:clean``` 定义伪目标，防止文件目录下有名为clean 的文件导致make clean失效.
