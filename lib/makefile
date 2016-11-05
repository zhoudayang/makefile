CXXFLAGS = -g -Wall -o2
OBJ = add.o sub.o

libmath($(OBJ)):$(OBJ)
	$(AR) $(ARFLAGS) libmath.a $(OBJ)

%.o:%.cc
	$(CXX) -c $(CXXFLAGS) $< -o $@

.PHONY:clean
clean:
	-rm -rf *.o *.a
