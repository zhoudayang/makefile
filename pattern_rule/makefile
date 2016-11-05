
CXXFLAGS = -g -Wall -o2 
TARGET = main
OBJ = main.o cal.o

.PHONY:all

all:$(TARGET) 

%.o:%.cc
	$(CXX) $(CXXFLAGS) -c $< -o $@

$(TARGET):$(OBJ)
	g++ -o $(TARGET) $(OBJ)

.PHONY:clean
clean:
	-rm -rf $(OBJ) $(TARGET)
