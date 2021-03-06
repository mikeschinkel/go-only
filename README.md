# only.Once

Tiny GoLang package to support use of the `for range only.Once {...}` construct. 

The idea is to use a GoLang `for range` loop that will **only ever execute once** to empower developers to use `break` to jump to the end of a lineary code sequence in contexts where many developers would where `break` would not normally work and thus where most developers would `return` early.

## Philosphy

The driving idea behind `only.Once` is that there should only ever be one return in a function, and it should be on the last line. 

## Usage Pattern

Here is a contrived error showing the usage pattern when using `only.Once`:

```golang
func example() (value Value, err error) {
   for range only.Once {
      value,err = funcMightError()
      if err != nil {
         err = fmt.Sprintf( "unable to do func: %s, err )
         break
      }
      value,err = anotherFuncMightError()
      if err != nil {
         err = fmt.Sprintf( "unable to do another func: %s, err )
         break
      }
   }
   return value,err
}
```

And here is another way we might write that same function _(Notice how we do work after the `break` but before the `return`):

```golang
func example() (value Value, err error) {
   var attempt string
   for range only.Once {
      value,err = funcMightError()
      if err != nil {
         attempt = "func"
         break
      }
      value,err = anotherFuncMightError()
      if err != nil {
         attempt = "another func"
         break
      }
   }
   if attempt != "" {
      err = fmt.Sprintf( "unable to do %s: %s, attempt, err )
   }
   return value,err
}
```

## Benfits

The immediate obvious benefits are:

1. There is only ever a single exit point in your functions.
2. You can still exit logic early using `break` similar to how you may have previously used `return` to exit early.
3. There is only ever one line of code you need to breakpoint in a debugging IDE to ensure execution does not continue past the function you are focusing on.
4. Unlike with early returns, you can run some shared code that is guaranteed to run no matter how the exit occurs.
5. Supports [_"The Happy_](https://medium.com/@matryer/line-of-sight-in-code-186dd7cdea88) [_Path,"_ e.g. left-aligning code](https://maelvls.dev/go-happy-line-of-sight/) _(albiet with one consistent level of indent)_

The longer term benefits are less obvious, but experience reveals them to be even more valuable:

1. Functions that have too many breaks need to be broken up is an obvious indicator they need to be split into two or more.
2. Functions that need nested `for range only.Once {}` constructs need to be broken up.
3. This establishes a very easy to remember and easy to write repeatable pattern for code, unlike w/early returns.
4. Using this pattern it become extremely easy to move logic to other functions during refactor w/o having to restructure logic

## Shortcomings
Given that this is a technique and not a built-in language feature:

1. It might be **infintismally slower** than an early `return`. So if every nanosecond matters in your executions you might be better to return early in the time critical sections of your code. However, I doubt you will even be able to measure the differences in most cases.

2. You might need to **explicitly declare variables** in contexts where you would not have to with early returns.

HOWEVER:

1. If the Go team were to add a dedicated construct to replace this technique and 
2. I Go 2.0 decided to drop block-specific variable scoping — which IMO causes far too many bugs and offers few to no real benefits — then you might no longer need to explicitly declare variables.

So, if after using this technique you recognize is has the same level of value that I do, please help me lobby the Go team to get rid of these two (2) shortcomings.

## Self-contained Example

Here is a contrived example showing how the `for range only.Once {}` construct is used in a few different contexts.

```
package main

import (
   "errors"
   "fmt"
   "github.com/mikeschinkel/go-only"
   "log"
   "math/rand"
   "time"
)

// main calls getValue() and prints either error or a message w/the value.
func main()  {
   for range only.Once {
      value,err := getValue()
      if err != nil {
         log.Fatalln(err)
      }
      fmt.Printf("Value is %d!",value)
   }
}

// getValue randomly returns a value from 2 to 10, but errors on 0 or 1.
func getValue() (v int, err error) {
   for range only.Once {
      v = getInt()
      if v == 0 {
         err = errors.New("Value cannot be 0. :-(")
         break
      }
      v = getInt()
      if v == 1 {
         err = errors.New("Value cannot be 1. :-(")
         break
      }
   }
   return v,err
}

// getInt randomly returns a value from 0 to 10.
func getInt() int {
   rand.Seed(time.Now().UnixNano())
   return rand.Intn(10)
}
````
