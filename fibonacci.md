# Fibonacci function in Go

I decided to write the Fibonacci function as my first non-trivial Go program.

```go
package main

import "fmt"

func fibonacci(n int) int {
    fibonacci_value := make([]int, n + 1)    // line A
    fibonacci_value[0] = 0
    fibonacci_value[1] = 1
	
    for index := 2; index < n + 1; index++ { // line B
        fibonacci_value[index] =             // line C
	    fibonacci_value[index - 1] +
	    fibonacci_value[index - 2]
    }

    return fibonacci_value[n]
}

func main() {
    fmt.Println(fibonacci(42))
}
```
																															  
* **line A**: Since the array size depends on our variable `n`, the usual array declaration cannot be used. Hence, we use the `make()` method to declare the array.
* **line B**: Go provides one and only one looping construct, the `for` loop. Unlike C, C++, Java, etc., the syntax disallows parenthesis here.
* **line C**: An explicit semicolon is not required after every statement in Go, since it inserts one implicitly, subject to some rules as described in [this StackOverflow answer](https://stackoverflow.com/a/34848928/252576). This also means long lines cannot be broken at arbitrary places.

The code prints `267914296` as expected.
