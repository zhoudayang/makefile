.PHONY : all
all: exec1 exec2

objects=exec1.o exec2.o cal.o
exec1_objs = exec1.o cal.o
exec2_objs = exec2.o cal.o
XX = g++
CXXFLAGS = -g -o2 -Wall

exec1:$(exec1_objs)
	$(XX) -o exec1 $(exec1_objs)

exec2:$(exec2_objs)
	$(XX) -o exec2 $(exec2_objs) 

$(objects):%.o:%.cpp
	$(XX) $(CXXFLAGS) -c $< -o $@

.PHONY:clean
clean:
	-rm -f $(objects) exec1 exec2

