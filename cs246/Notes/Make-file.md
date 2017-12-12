# Make file
```shell
g++14 *.cc -o prog
```
- Inefficient, always (re)compiles all .cc files

### Separate Compilation
```shell
g++14 -c main.cc
g++14 -c book.cc
g++14 -c comic.cc
g++14 -c textbook.cc
g++14 *.o -o prog
```
- We now only need to recompile files that have changed.
- We need to keep track of the files that changed.
    - A very long string of commands.

```shell
main: main.o book.o textbook.o comic.o // dependencies
[TAB] g++ -std=c++14 main.o book.o textbook.o comic.o -o main
    
book.o: book.h book.cc
    g++ -std=c++14 -c book.cc
    
comic.o: comic.h comic.cc book.h
    ...
```
    