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

# Chapter 1: Set up Go Environment

## Go Workspace
```
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```
Use `go env` to show all the environment variables

Environmemnt variable:

+ variable `GOPATH`: point to your go workspace

+ variable `GOROOT`: point to your binary installation of GO
## Go Command

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



# Chapter 2: Primitive Types and Declarations

## Build-in Types
booolenas, integers, floats, strings
### Zero value
default zero value to any variable that is declared but not assigned a values.
### Literals
0b binary

0o octal (eight)

0x hexadecimal (sixteen)

For longer integer literals, use underscores to improve readability `1_234`

## var Versus :=
`var Versus :=`

## Using const

`const x int64 = 10`

`const y = "hello"`

## Unused Variables

Every declared local variable must be used.

```
func main() {
    x := 10
    x = 20
    fmt.Println(x)
    x = 30
}
// valid
```

Go compiler won't stop you from creating unread package-level variables (golangci-lint could detect them).

Go allows you to create unread constants with `const`.

## Naming Variables and Constants

Go requires identifier names to start with a letter or underscore, and the name can contain numbers, underscores, and letters.

Idiomatic go doesn't use snake case (names like `index_counter` or `number_tries`). Instead, go use camel case (names like `indexCounter` or `numberTries`).

Go uses the case of the first letter in the name of a package-level declaration to determine if the item is accessible outside the package.

Within a function, favor short variable names. 
The smaller the scope for a variable,
the shorter the name that’s used for it. 
It is very common in Go to see single-lette variable names.
- The names `k` and `v` (short for key and value) are used as the variable names in a for-range loop.
- If you are using a standard for loop, `i` and `j` are common names for the index variable.
- Provide variable types and single-letter names. For example, `i`for integers, `f` for floats, `b` for booleans.

For variables and constants in package block, we need to use more descriptive names.

# Chapter 3: Composite Types
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
x := make([]int, 5)
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

```
x := []int{1, 2, 3, 4}
y := x[:2]
z := x[1:]
d := x[1:3]
e := x[:]
```

When take a slice from a slice, you are *not* making a copy of the date. You have two variables that are sharing memory.

Changes to an element in a slice affect all slices share that element. NEVER use `append` with subslices.

Use three-part slice expression to limit capacity that is available for the subslice.

### Covert arrays to slices

Take a slice from array using a slice expression
```
x := [4]int{5, 6, 7, 8}
y := x[:2]
```

Have the same memory-sharing issue.

### Copy

Use copy built-in function to create a slice that is independent of the original.

```
num := copy(destination_slice, source_slice)
```

`num` is the number of elements copoed, limited by whichever slice is smaller (length). If do not need `num`, do not need to assign it.

## Strings and Runes and Bytes

String is composed of a sequence of UTF-8-encoded code points.
> UTF-8: most commonly used encoding for Unicode
> 
> Unicode uses foru bytes (32 bits) to represent each code point (code point is the technical name for each character and modifier) : UTF-32 (waste so much space)
> 
> UTF-16 (wasteful)
> 
> UTF-8: allow you to look at any byte in a sequence and tell if you are at the start of a UTF-8 sequence, or somewhere in the middle.


Slice substrings. `s[2]` and `s[2:]`
- A string is composed of a sequence of bytes, while a code point in UTF-8 can be anywhere from one to four bytes long. (such as emoji)
- Only use slicing when you know that your string only contains chars that take up one byte.
- Instead of using slice and index expressions, try to use functions in `strings` and `unicode/utf8` packages to iterate over the code points in a string. (*next chapter*)

`len(s)`

Covert a single rune or byte to a string
```
var a rune = 'x'
var s string = string(a)
var b byte = 'y'
var s2 string = string(b)
```
Cannot convert an int into a string
```
var x int = 65
var y = string(x)
fmt.Println(y) // do not print 65, but A
```

## Maps
Declare map: `map[keyType]valueType`
```
var nilMap map[string]int // declare nil map
totalWins := map[string]int{} // declare empty map
ages := make(map[int][]string, 10)
```

- Maps are not comparable. You can check if a map equal to nil, but you cannot check if two maps have identical keys and values using `==` or `!=`
- Know how many key-values pairs, using `make` to create a map
- `len(map)`
- The key for a map must be comparable type (cannot use slice or map)
- The type of the value can be anything

> Hash Map
>
> When insert a key and value, the key is turned into a number using a *hash algorithm*
>
> When two keys map to the same bucket, cause collision.
>
> Go does not require (allow) you to define your own hash algorithm.

### Reading and Writing a Map

```
totalWins := map[string]int{}
totalWins["Orcas"] = 1
totalWins["Lions"] = 2
fmt.Println(totalWins["Orcas"])
fmt.Println(totalWins["Kittens"]) // 0
totalWins["Kittens"]++
fmt.Println(totalWins["Kittens"])
totalWins["Lions"] = 3
fmt.Println(totalWins["Lions"])
```

When we try to read the value assigned to a map key that was never set, the map returns the *zero* value for the map’s value type.

### The comma ok idiom

Comma ok idiom tells the difference between a key that's associated with a zero value and a key that is not in the map.

```
m := map[string]int{
"hello": 5,
"world": 0,
}
v, ok := m["hello"]
fmt.Println(v, ok) // 5, true
v, ok = m["world"]
fmt.Println(v, ok) // 0, true
v, ok = m["goodbye"]
fmt.Println(v, ok) // 0, false
```

### Deleting from Maps
```
m := map[string]int{
"hello": 5,
"world": 10,
}
delete(m, "hello")
```

### Using Maps as Sets

Set is a data type that ensures there is at most one of a value, but not guarantee the values are in any particular order.

Go does not include a set, but we can use a map to simulate some of its features. 
Create a map where the keys are of int type and the values are of bool type.

```
intSet := map[int]bool{}
vals := []int{5, 10, 2, 5, 8, 7, 3, 9, 1, 2, 10}
for _, v := range vals {
    intSet[v] = true
}
fmt.Println(len(vals), len(intSet))
fmt.Println(intSet[5])
fmt.Println(intSet[500])
if intSet[100] {
    fmt.Println("100 is in the set")
}
```

## Structs

For maps, 
- They don’t define an API since there’s no way to constrain a map to only allow certain
keys
- All values in a map must be of the same type

```
type person struct {
    name string
    age int
    pet string
}
```
- There are no commas separating the fields in a struct
declaration.
- A struct type that’s defined within a function can only be used within that function.

```
var fred person
bob := person{}
// No difference for the above two methods
```

Since no value is assigned to `fred`, it gets the zero value for the `person` struct type. A zero value struct has every field set to the
field’s zero value.

```
julia := person{
    "Julia",
    40,
    "cat",
}
```
A struct literal can be specified as a comma-separated list of values for the fields inside of braces (in order). Used for simple struct

```
beth := person{
age: 30,
name: "Beth",
}
```
Map literal style: you can leave out keys and specify the fields in any order. Used for complex struct.

```
bob.name = "Bob"
fmt.Println(beth.name)
```
Dotted notation.

### Anonymous Structs

```
var person struct {
    name string
    age int
    pet string
}

person.name = "bob"
person.age = 50
person.pet = "dog"

pet := struct {
    name string
    kind string
}{
    name: "Fido",
    kind: "dog",
}
```
Usages for anonymous struct:
- Translate external data into a struct or a struct into external data (like JSON or protocol buffers) (called unmarshaling and marshaling
data)
- Anonymous structs pop up: writing table-driven tests

### Comparing and Converting Structs
Whether or not a struct is comparable depends on the struct’s fields.
Structs consists slice or map fields (and function and channel) fields are not comparable.

Go doesn’t allow comparisons between variables that represent structs of different types.

Go allow type conversion from one struct type to another if the fields of both structs have the same names, order, and types.

```
type firstPerson struct {
    name string
    age int
}

type secondPerson struct {
    name string
    age int
}

type thirdPerson struct {
    age int
    name string
}

type fourthPerson struct {
    firstName string
    age int
}

type fifthPerson struct {
    name string
    age int
    favoriteColor string
}
```

```
person := firstPerson{
	name: "Bob",
	age:  50,
}

person2 := secondPerson(person)

fmt.Println(person2)
fmt.Printf("%T", person2) // secondPerson
fmt.Printf("%T", person) //firstPerson
fmt.Println(person2 == person)// error: mismatched types secondPerson and firstPerson
```

- Can use a type conversion to convert an instance of `firstPerson` to
`secondPerson`
    - How to convert? `person := firstPerson{} person2 = secondPerson(person)`?
- Cannot use `==` to compare an instance of `firstPerson` and an instance of `secondPerson` (because they are different types)
- Cannot convert an instance of `firstPerson` to `thirdPerson`, because the fields are in a different order
- Cannot convert an instance of firstPerson to fourthPerson because the field names don’t match
- Cannot convert an instance of firstPerson to fifthPerson because there’s an additional field

```
type firstPerson struct {
	name string
	age  int
}
f := firstPerson{
	name: "Bob",
	age:  50,
}
var g struct {
	name string
	age  int
}
// compiles -- can use = and == between identical named and anonymous structs
g = f
fmt.Println(f == g)
```
If two struct variables are being compared and at least one of them has a type that’s an anonymous struct, 
you can compare them without a type conversion if the fields of both structs have the same names, order, and types. 
You can also assign between named and anonymous struct types if
the fields of both structs have the same names, order, and types
# Chapter 4: Blocks, Shadows, and Control Structures

# Chapter 13: Writing Tests

## The Basics of Testing
Go tests are placed in the same directory and the same package as the production code.

```
func addNumbers(x, y int) int {
    return x + x
}

func Test_addNumbers(t *testing.T) {
    result := addNumbers(2,3)
    if result != 5 {
        t.Error("incorrect result: expected 5, got", result)
    }
}
```

- Every test file ends with `_test.go`
- Test functions name `Test` + document/function what you are testing
- Test functions take in a single parameter of type `*testing.T`. By convention, this parameter is named `t`
- When there is an incorrect result, we report the error with `t.Error` method.
- `go test ./...`

### Reporting Test Failures

Error/Errorf: if test several independent items. So that can report as many problems at once.

Fatal/Fatalf: test function exits immediately after the test failure message is generated. If the failure of a check in a test means that further checks in the same test function will
always fail or cause the test to panic.

### Setting up and tearing down

```
func TestMain(m *testing.M) {
    fmt.Println("Set up stuff for tests here")
    testTime = time.Now()
    exitVal := m.Run()
    fmt.Println("Clean up stuff after tests here")
    os.Exit(exitVal)
}
```

- Parameter of type `*testing.M`
- Running go on a package with a `TestMain` function calls the function instead of invoking the tests directly.
- Once the state is configured, call the `Run` method on `*testing.M` to run the test functions. 
  - The `Run` method returns the exit code; 
  - `0` indicates that all tests passed. 
- Finally, you must call `os.Exit` with the exit code returned from `Run`.
- `TestMain` is invoked once, not before and after each individual test
- you can have only one `TestMain` per package.

Using `Cleanup` method or `defer` statement to clean up temp resources created for a single test.

### Storing Sample Test Data
Create a subdirectory named `testdata` to hold sample data to test functions.

### Caching Test Results

Go also caches test results when running tests across multiple packages if they have passed and their code hasn’t changed.
- using `count=1` flag to force tests akways run

### Testing your Public API
TODO

### Use go-cmp to Compare Test Results
TODO

## Table Tests
TODO

## Checking Your Code Coverage

```
go test -v -cover -coverprofile=c.out
go tool cover -html=c.out
```

- `-cover`: calculate coverage information and include a summary in the test ourtput
- `-coverprofile`: save the coverage information to a file

