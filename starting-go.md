# Starting Go

I decided to learn Go. So, here I start with the obligatory Hello World program:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello World")
}
```

The entry point of a Go program is the `main()` function in the main package. The `"fmt"` package is imported here to enable I/O operations. The `Println()` method prints the string.

Some setup is (obviously) required to compile and execute this program. I downloaded the Windows version of the Go setup and executed it with default options. For now, I am interested in the compiler, which is at `C:\Go\bin\go.exe`.

The Go program is compiled with the following command:

> go build &lt;filename&gt;

The generated executable gives the desired output on execution:

> Hello World

Now, I hope to write many more Go programs, so to make this execution easier, I configure the NppExec plugin of Notepad++ with this script:

```
NPP_SAVE
CD $(CURRENT_DIRECTORY)
C:\Go\bin\go build "$(FILE_NAME)"
$(NAME_PART).exe
```
