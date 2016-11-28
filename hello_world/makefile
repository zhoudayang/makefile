main:main.o
	c++ -o main main.o
main.o:main.cc
	c++ -c main.cc -o main.o
.PHONY : clean
clean:
	-rm -rf *.o main
