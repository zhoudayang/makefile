
CPPFLAGS = -g -Wall -o2 
TARGET = main
OBJ = main.o cal.o

.PHONY:all

all:$(TARGET) 

$(OBJ):$(pubsubst %.o,%.cc,$(OBJ))

$(TARGET):$(OBJ)
	g++ -o $(TARGET) $(OBJ)

.PHONY:clean
clean:
	-rm -rf $(OBJ) $(TARGET)
