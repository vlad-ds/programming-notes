# Make

**Make** is a [build automation](https://en.wikipedia.org/wiki/Build_automation) tool that automatically [builds](https://en.wikipedia.org/wiki/Software_build) [executable programs](https://en.wikipedia.org/wiki/Executable_program) and [libraries](https://en.wikipedia.org/wiki/Library_(software)) from [source code](https://en.wikipedia.org/wiki/Source_code) by reading [files](https://en.wikipedia.org/wiki/File_(computing)) called [Makefiles](https://en.wikipedia.org/wiki/Makefile) which specify how to derive the target program (Wikipedia).

Here is the syntax of a typical rule:

```
target: prerequisites
<TAB> recipe
```

A simple Makefile:

```makefile
say_hello:
        @echo "Hello World"
generate:
        @echo "Creating empty text files..."
        touch file-{1..10}.txt
clean:
        @echo "Cleaning up..."
        rm *.txt
```

Run it by typing `make`.  The `@` ensures that the command is not displayed when running make, just its output. 

`say_hello`, `generate` and `clean` are targets: but they are phony targets because they are just a name. The targets are followed by prerequisites or dependencies. The recipe uses prerequisites to make a target. Together, target prerequisites and recipes make a *rule*. 

By default, only the first target in the makefile is the default target. In most projects, `all` is the first target. Its responsibility is to call other targets. 

```makefile
.PHONY: all say_hello generate clean
all: say_hello generate

say_hello:
        @echo "Hello World"

generate:
        @echo "Creating empty text files..."
        touch file-{1..10}.txt

clean:
        @echo "Cleaning up..."
        rm *.txt
```

The special phony target `.PHONY` is used to define all the targets that are not files. So `make` will run regardless of whether a file with that name exists. 



