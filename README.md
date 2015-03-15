godebug
-------

A debugger for Go.

### How it works

`godebug` uses source code generation to instrument your program with debugging calls. [go tool cover](http://blog.golang.org/cover) takes a similar approach to code coverage. When you run `godebug`, it parses your program, instruments function calls, variable declarations, and statement lines, and outputs the resulting code somewhere (currently either stdout or in place over your original files). When you run this modified code, assuming you put a breakpoint somewhere, you can step through it and inspect variables. Coming later: evaluate arbitrary Go expressions and write to variables.

For more detail, see the end of this README.


### Status

`godebug` is currently in alpha stage -- expect some problems.


### Installation:

    $ go get github.com/mailgun/godebug


### Use:

First, get your directory in a clean state. **The command below will overwrite your files, so make sure you have committed or stashed everything.**

In any file where you want a breakpoint, import `github.com/mailgun/godebug/lib` and insert this breakpoint anywhere in the code: `godebug.SetTrace()`. Then run:

    $ godebug -w .

Your code is now self-debugging. Congrats! Run it with `go run`, test it with `go test`, or build then run it with `go build`.


### Debugger commands:

The current commands are:

command       | result
--------------|------------------------
h(elp)        | show help message
n(ext)        | run the next line
s(tep)        | run for one step
c(ontinue)    | run until the next breakpoint
l(ist)        | show the current line in context of the code around it
p(rint) [var] | print a variable

The debugger will attempt to interpret any text that does not match the above commands as a variable name. If that variable exists, the debugger will print it.

### How it works (more detail)

Consider this program:

    package main

    import "fmt"

    func main() {
        fmt.Println("Hello, world!")
    }

Now let's modify it a bit:

    package main

    import (
        "bufio"
        "fmt"
        "os"
    )

    func main() {
        fmt.Println(`-> fmt.Println("Hello, world!")`)
        bufio.NewScanner(os.Stdin).Scan()
        fmt.Println("Hello, world!")
    }

When we run this modified version, we see:

    -> fmt.Println("Hello, world!")

And then the program waits for input before proceeding.

We have just implemented a debugger for the first program! It may not seem like much, but this program implements two fundamental debugger behaviors: (1) display the the current state of the program, and (2) do not proceed until instructed by the user. Furthermore, the changes we made were straightforward and easy to automate:

  * insert import statements for `bufio` and `os`, if not already present.
  * in `main()`, insert the statement `fmt.Println(<quote next line>)`
  * in `main()`, insert the statement `bufio.NewScanner(os.Stdin).Scan()`.

We could do exactly the same thing for any other program with a single-line main function. And it's not hard to see how to generalize this to multiple lines. This, in essence, is what godebug does. Parse source code, insert extra code that implements the behavior of a debugger for that program, output and run the result. godebug handles many more cases than this simple example and implements more interesting debugging behavior, but the principle is exactly the same.
