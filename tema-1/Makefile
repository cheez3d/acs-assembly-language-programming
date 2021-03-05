CFLAGS=-m32
AFLAGS=-f elf

build: tema1

tema1: tema1.o includes/ASTUtils.o includes/macro.o
	gcc -g $^ -o $@ $(CFLAGS)

tema1.o: tema1.asm
	nasm -g $^ -o $@ $(AFLAGS)

run: tema1
	./tema1 <input.txt

clean:
	rm -rf tema1.o tema1
