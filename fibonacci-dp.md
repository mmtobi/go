# Dynamic Programming with Go: Fibonacci function

To gain more experience with Go, I chose to do some Dynamic Programming (DP) using Go. 

## Naive Recursive Implementation

I use the classic Fibonacci function in this example.  Let's begin with the naive recursive approach.

```go
package main

import "fmt"    // Use the fmt package to print the output

func fib_naive(n int) int {
    var value int   // Syntax to declare a variable `value` of type int
    if n == 0 {
        value = 0   // We could avoid the variable `value` by returning the result directly
                    // However, we write the code this way for easy transformation to DP versions
    } else if n == 1 {
        value = 1
    } else {
        value = fib_naive(n - 1) + fib_naive(n - 2)
    }
    return value
}

func main() {
    fmt.Println("Fibonacci value=", fib_naive(42))
}
```

This prints the expected value:

> Fibonacci value= 267914296

Of course, the naive recursive implementation has exponential time complexity. 

## Memoized Implementation: first attempt

Next, I move on to the top-down DP implementation using memoization.
The `main()` method and other ceremony is omitted to reduce clutter.

```go
// WARNING: This implementation is faulty, read on further for the corrected implementation
func fib(n int) int {
    var value int
    memo := map[int]int{}   // Declares an empty map (memo) to memoize the Fibonacci values
    if _, exists := memo[n]; exists { 
            // if statement in Go can have an initialization and a condition
            //     (side-note: C++17 adapts this feature for `if` and `while` constructs)
            // Placeholder _ is used to ignore one or more returned values
            //     (side-note: Structured binding in C++17 doesn't allow ignoring return values)
            // Here this is done for demo. The returned value will be used in the next version below.
        return memo[n]  // If the value already exists in the memo, return it immediately 
    } else if n == 0 {  // Rest of this if else ladder same as naive recursive implementation
        value = 0
    } else if n == 1 {
        value = 1
    } else {
        value = fib(n - 1) + fib(n - 2)
    }
    memo[n] = value     // Enter the value into the memo
    return memo[n]
}
```

The naive recursive implementation is transformed to the memoized version using a couple of code changes:

* Check for the required value in the memo first, and return it immediately if available.
* Enter the computed value in the memo the first time we compute it. 

Logically, this cuts down the exponential time complexity to linear time because each Fibonacci value is computed only once. 

## Using the `time` package 
To verify this complexity reduction, let us compare the execution times. To do this, we include the `time` package.

The package import section now looks like this:

```go
import (     // Import multiple packages using this parenthesis syntax
             // Specifying import separately for each package also works
    "fmt"    // Use the fmt package to print the output
    "time"   // Use the time package for calculating execution times
)
```

The `main()` method now looks like this:

```go
func main() {
    start_time := time.Now()    // returns a `time.Duration` type object
    fmt.Println("Fibonacci value=", fib_naive(42))
    naive_end := time.Now()
    fmt.Println("Naive implementation execution time=",
        naive_end.Sub(start_time))  // Use the `Sub()` method to calculate time difference

    fmt.Println("Fibonacci value=", fib(42))
    memo_end := time.Now()
    fmt.Println("Memoized implementation execution time=",
        memo_end.Sub(naive_end))
}
```

The output is as follows:

> Fibonacci value= 267914296<br>
> Naive implementation execution time= 5.3756892s<br>
> Fibonacci value= 267914296<br>
> Memoized implementation execution time= 1m5.5506087s

Clearly, that didn't work as expected. The memoized implementation takes *much longer* than the naive recursive implementation. What went wrong there?

The problem is that each call to the memoized method `fib()` creates its own `memo` map! In C/C++, this could be solved by using a `static` local variable which would retain its value across multiple function calls. However, Go doesn't have anything like a `static` variable, so we need to move the `memo` outside the function. 

## Memoized Implementation: second attempt (with Lambda function and Closure)

We could achieve this by declaring `memo` as a "global" variable, but I take a better approach: closures and lambda functions. Now, the memoized implementation looks like below:

```go
func fib_memo(n int) int {  // The "outer" function
    memo := map[int]int{}   // memo is now in a closure
    var fib func(n int) int // Declare the "inner" function so that it can be called recursively
    
    fib = func(n int) int { // The "inner" lambda function
        var value int
        if fib_value, exists := memo[n]; exists {   
                // We cannot "reuse" the variable `value` declared above
                // to assign the value of `memo[n], since the variable 
                // declared here would be local to the ``if` scope
            return fib_value
        } else if n == 0 {
            value = 0
        } else if n == 1 {
            value = 1
        } else {
            value = fib(n - 1) + fib(n - 2) // (Memoized) recursive call to the lambda function
        }
        memo[n] = value
        return memo[n]
    }
    
    return fib(n)   // Call the lambda function to get the Fibonacci value
}
```

The complexity is now reduced to linear time, and the execution time is too small to be observed!

> Fibonacci value= 267914296<br>
> Memoized implementation execution time= 0s

## Decluttering the Implementation

We can now declutter this code a little by initializing the map with the first two values. The `fib_memo()` method now becomes:

```go
func fib_memo(n int) int {
    memo := map[int]int {
        0: 0,   // Initialize the map with Fibonacci values for 0 and 1
        1: 1,   // Comma after the last key-value pair is required.
    }
    var fib func(n int) int
    
    fib = func(n int) int {
        if fib_value, exists := memo[n]; exists {
            return fib_value
        } else {
            memo[n] = fib(n - 1) + fib(n - 2)
            return memo[n]
        }
    }
    
    return fib(n)
}
```

Further, by observing that the returned value in both `if` and `else` part is essentially `memo[n]`, and switching the order of `if` and `else` blocks, this code can be further decluttered:

```go
func fib_memo(n int) int {
    memo := map[int]int {
        0: 0,   // Initialize the map with Fibonacci values for 0 and 1
        1: 1,   // Comma after the last key-value pair is required.
    }
    var fib func(n int) int
    
    fib = func(n int) int {
        if _, exists := memo[n]; !exists {
            // If the Fibonacci value doesn't exist already in the memo, 
            // compute it via memoized recursive call and add it to the memo
            memo[n] = fib(n - 1) + fib(n - 2)
        }
        return memo[n]
    }
    
    return fib(n)
}
```

## Bottom-up DP implementation

Next, I implement the bottom-up DP:

```go
func fib_dp(n int) int {
    var fib_value = make(map[int]int, n + 1) 
        // Use `make()` to allocate capacity for `n + 1` pairs to avoid map resizing.
        //      (side-note: This is similar to reserve() methods of C++ containers. 
        //                  The map _can_ grow beyond the specified capacity.)
        // Note: This optimization could have been done in the `fib_memo()`'s map too.
    fib_value[0] = 0
    fib_value[1] = 1
    
    for k := 2; k <= n; k++ {
            // Go has a single looping construct, the for loop.
            // There's also no "preincrement" unlike C/C++.
        fib_value[k] = fib_value[k - 1] + fib_value[k - 2]
    }
    
    return fib_value[n]
}
```
