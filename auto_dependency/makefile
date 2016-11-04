OBJS=cal.o main.o
TARGET=main
CFLAGS=-g -o2 -Wall
XX = g++

$(TARGET):$(OBJS)
	$(XX) $(OBJS) -o $(TARGET)

$(OBJS):%.o:%.cpp
	$(XX)  -c $(CFLAGS)  $< -o $@

.PHONY: clean
clean:
	rm $(OBJS) $(TARGET)

