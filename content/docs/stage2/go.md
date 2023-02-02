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

## Go Workspace
```
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```
Use `go env` to show all the environment variables

Environmemnt variable:

+ variable `GOPATH`: point to your go workspace

+ variable `GOROOT`: point to your binary installation of GO
## go Command

`go run` compile your code into a binary. However, the binary is build in a temporary directory.

`go build` 
+ `go build -o` + binary name

### Getting third party go tools
`go install location_of_source_code_repo@version`: 
Install the binary in `$GOPATH/bin` directory.

Update tools to a newer version: `go install location_of_source_code_repo@newer_version`

### Formatting your code
`go fmt` automatically reformats your code to match the standard format, including indentation, white space, spacing around operators.

`goimports` is an enhanced version of `go fmt`

Semicolon insertion rule to faster compile and enforce coding style.

## Linting and Vetting

`golint`: ensure code following style guidelines. Includes properly naming variables, formatting error messages, and placing comments on public methods and types. (are not errors)
```
go install golang.org/x/lint/golint@latest
golint ./...
```

`go vet` for passing the wrong number of parameters to formatting methods or assigning values to variables that are never used. `go ver ./...` for entire project

`golangci-lint`: combine golint, go vet and others. `golangci-lint run`
- `.golangci.yml` at the root of your project: configure which linters are enabled, which files they analyze. 
- https://golangci-lint.run/usage/configuration/

## Makefile

```
.DEFAULT_GOAL := build

fmt:
    go fmt ./...
.PHONY:fmt

lint: fmt
    golint ./...
.PHONY:lint

vet: fmt
    go vet ./...
.PHONY:vet

build: vet
    go build hello.go
.PHONY:build
```

`.DEFAULT_GOAL`: which target is run when no target is specified.

`name_of_target: targets_must_be_run_before_the_target`

The `.PHONY` line keeps make from getting confused if you ever create a directory in your project with the same name as a target.


## Staying Up tp Date

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

`go get` package management

```
mkdir go_programming
cd go_programming
go mod init example.com/test
go env -w GO111MODULE=on
go get -d github.com/GoesToEleven/GolangTraining/...
```
Dowload them in /root/go/pkg/...

`go install xxx` will install package in /root/go/bin



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

Cannot change the size of an array, since length is part of the type the array.

## Slices

```
var x []int
var x [][]int
```

`nil`: an identifier that represents the lack of a value for some types

slices is not comparable. (can use `reflect.DeepEqual` to compare)
```
var x []int
var y []int
fmt.Println(x == y) // compile-time error
fmt.Println(x == nil) // print true
len(x) // return 0
x = append(x, 10, 5, 7)
var z = []int{1, 2, 3}
x= append(x, z...)
```

>Why need **x** = append(x, 10)? 
>
>Go is a call by value language. Every time you pass a parameter to a function, Go makes a copy of the value that’s passed in. Passing a slice to the append function actually passes a copy of the slice to the function. The function adds the values to the copy of the slice and returns the copy. You then assign the returned slice back to the variable in the calling function.

### Capacity
capacity: the number of consecutive memory locations reserved.

When the length reaches the capacity, there’s no more room to put values. 
If you try to add additional values when the length equals the capacity, the function append uses the Go runtime to allocate a new slice with a larger capacity (usually double the size of the slice, or grow by at least 25%). 
The values in the original slice are copied to the new slice, the new values are added to the end, and the new slice is returned.

>Go Runtime
>
> + Go runtime is the software stack responsible for building and running your code.
> + Go runtime is compiled into every go binary
> + Go runtime provides services like memory allocation and garbage collection, concurrency support, networking, and implementations of built-in types and functions.

`cap(x)` return the current capacity of slice `x`

### make slices
```
x = make([]int, 5)
x = append(x, 10) // [0,0,0,0,0,10]
```
`append` always inceases the length of a slice.

### Declaring your Slice
Goal: minimize the number of times the slice need to grow

+ declaring a slice that might stay nil: `var date []int`
+ declaring a slice with default values: `date = []int{1, 2, 3}`
+ know how large your slice needs to be, but do not know what those values: using `make`
    + use slice as buffer: nonzero length
    + know the exact size you want: specify the length and index into the slice to set the values
    + others: **zero length**, specified capacity. allow use `append` to add items

### Slicing Slices