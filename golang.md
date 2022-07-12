# Go

[Writing Web Applications](https://go.dev/doc/articles/wiki/#tmp_4)

- Get localhost for Linux:     
  `ifconfig | grep "inet " | grep -v 127.0.0.1`
- Using [inet]:[port]/...

## Why Go for backend
Built-in concurrency mechanism

Go has no virtual machinee or compiles to machine code, so programs are execute fast without warm-up time like Java.

Also, Go has a built-in garbage collector that monitors and identifies occupied memory that’s no longer needed and frees it for reuse. This lowers the risk of security vulnerabilities due to code encapsulation and provides effective memory management. 

Go is an ideal solution for building microservices due to the tiny memory footprint and fast performance of Go apps as well as the convenience of statically linked binaries.

## Go syntax
[Go cheatsheet](https://quickref.me/golang)

Go's basic types are
```
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
```

## Go Basic Example

```Go
package main

import (
	"fmt"
	"math/rand"
)

var c, python, java bool

var i, j int = 1, 2

var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

// can be shortened to x, y int
func add(x int, y int) int {
	return x + y
}

// can return any number of results
func swap(x, y string) (string, string) {
	return y, x
}


func main() {
  // The environment in which these programs are executed is deterministic => rand.Intn will return the same number
  // To see a different number, seed the number generator: rand.Seed
  fmt.Println("My favorite number is", rand.Intn(10))

  fmt.Printf("Now you have %g problems.\n", math.Sqrt(7))

  // Exported names begin with a capital letter
  fmt.Println(math.Pi)
  
  fmt.Println(add(42, 13))
  
  a, b := swap("hello", "world")
  fmt.Println(a, b)
  
  // The var statement declares a list of variables
  // Default int is 0, boolean is false, "" (the empty string) for strings
  var i int
  fmt.Println(i, c, python, java)
  
  var c, python, java = true, false, "no!"
  fmt.Println(i, j, c, python, java)
  
  // Inside a function, the := short assignment statement can be used in place of a var declaration with implicit type.
  var i, j int = 1, 2
  k := 3
  c, python, java := true, false, "no!"
  fmt.Println(i, j, k, c, python, java)
  
  fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
  fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
  fmt.Printf("Type: %T Value: %v\n", z, z)
  
  const Truth = true
  fmt.Println("Go rules?", Truth)
  
  sum := 0
  for i := 0; i < 10; i++ {
    sum += i
  }
  fmt.Println(sum)
  
  // array
  var a [2]string
  a[0] = "Hello"
  a[1] = "World"
  fmt.Println(a[0], a[1])
  fmt.Println(a)

  primes := [6]int{2, 3, 5, 7, 11, 13}
  fmt.Println(primes)
  
  // This selects a half-open range which includes the first element, but excludes the last one.
  // Slices are like references to arrays
  var s []int = primes[1:4]
  fmt.Println(s)
  
  // The zero value of a slice is nil.
  var s []int
  fmt.Println(s, len(s), cap(s))
  if s == nil {
    fmt.Println("nil!")
  }
	
  // The make function allocates a zeroed array and returns a slice that refers to that array:
  a := make([]int, 5)  // len(a)=5
  // To specify a capacity, pass a third argument to make:
  b := make([]int, 0, 5) // len(b)=0, cap(b)=5
}
```

