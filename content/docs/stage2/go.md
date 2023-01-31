---
title: "Go"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Set up GO Environment

Install a secondary Go environment
```
go get golang.org/dl/go.1.15.6
go1.15.6 download
go1.15.6 build
```
Remove the secondary Go environment
```
go1.15.6 env GOROOT
rm -rf $(go1.15.6 env GOROOT)
rm $(go env GOPATH)/bin/go1.15.6
```

# Primitive Types and Declarations

## Build-in Types
booolenas, integers, floats, strings
### Zero value
default zero value to any variable that is declared but not assigned a values.
### Literals
0b binary

0o octal (eight)

0x hexadecimal (sixteen)

For longer integer literals, use underscores to improve readability `1_234`

`var Versus :=`

`const x int64 = 10`

`const y = "hello"`

Every declared local variable must be used.
# Composite Types
## Arrays: Too rigid to use directly
```
var x1 [3]int // x[0], x[1] and x[2] are initialized to zero.
var x2 = [3]int{10, 20, 30}
var x3 = [12]init{1, 5:4, 6, 10: 100, 15} // [1,0,0,0,0,4,6,0,0,0,100,15]
var x4 = [...]int{10, 20, 30}
fmt.Println(x == y) // Ture
fmt.Println(len(x))

var x [2][3]int // multidimensional arrays
```

Cannot use a negative index. e.g., `x[-1] = 10`

Cannot change the size of an array.


