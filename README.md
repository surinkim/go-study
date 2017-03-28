
Initialization
===========
#### Constants

 - They are created at compile time
 - They can only be numbers, characters (runes), strings or booleans. 
 - Because of the compile-time restriction, the expressions that define them must be constant expressions, evaluatable by the compiler. 


< - [code 1](https://play.golang.org/p/ohyi3NvqQg)

```golang
const age int  = 10
const x, y int = 30, 50

const (
	width, height int = 30, 50      
	no, name          = 10, "Maria"
)
```

 - iota

< - [code 2](http://www.cplusplus.com/reference/numeric/iota/)

```c++
//c++ code
int numbers[10];
std::iota (numbers,numbers+10,100);

```

   < - [code 3](https://play.golang.org/p/Sbwy_LVLtZ)


#### Variables
Variables can be initialized just like constants but the initializer can be a general expression computed at run time.

<- code 4

```golang
var (
    home   = os.Getenv("HOME")
    user   = os.Getenv("USER")
    gopath = os.Getenv("GOPATH")
)
```



#### The init function

 - `init` is called after all the variable declarations in the package have evaluated their initializers
 - A common use of init functions is to verify or repair correctness of the program state before real execution begins.

<- [code 5](http://stackoverflow.com/questions/24790175/when-is-the-init-function-in-go-golang-run)



Methods
=======

#### Pointers vs. Values
 - A methods can be defined for any named type (except a pointer or an interface); the receiver does not have to be a struct.

<- code 6
```go
func Append(slice, data []byte) []byte {
}
//can convert to below code
//  |
//  |
//  V
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
}

```

<- code 7
```go
func (p *ByteSlice) Append(data []byte) {
    slice := *p
    // Body as above, without the return.
    *p = slice
}
```

<- code 8
```go
func (p *ByteSlice) Write(data []byte) (n int, err error) {
    slice := *p
    // Again as above.
    *p = slice
    return len(data), nil
}
```

 - We pass the address of a `ByteSlice` because only `*ByteSlice` satisfies `io.Writer`. 
 - The rule about pointers vs. values for receivers is that value methods can be invoked on pointers and values, but pointer methods can only be invoked on pointers.
 - This rule arises because pointer methods can modify the receiver; invoking them on a value would cause the method to receive a copy of the value, so any modifications would be discarded. The language therefore disallows this mistake.

<- [code 9](https://play.golang.org/p/uFAl2BlhUy)


There is a handy exception, though. When the value is addressable, the language takes care of the common case of invoking a pointer method on a value by inserting the address operator automatically.

<- [code 10](https://play.golang.org/p/ragem2G-Ca)


Interfaces and other types
=====================

#### Interfaces
 - Interfaces in Go provide a way to specify the behavior of an object
 - [Fprintf](https://golang.org/pkg/fmt/#Fprintf)  can generate output to anything with a `Write` method. 
 - Usually given a name derived from the method, such as `io.Writer` for something that implements `Write`.
 - A type can implement multiple interfaces.
 
 <-code 11
```go
type Sequence []int

// Methods required by sort.Interface.
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// Method for printing - sorts the elements before printing.
func (s Sequence) String() string {
    sort.Sort(s)
    str := "["
    for i, elem := range s {
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}
```

#### Conversions
<-code 12
```go
func (s Sequence) String() string {
    sort.Sort(s)
    return fmt.Sprint([]int(s))
}
```
-  The conversion doesn't create a new value, it just temporarily acts as though the existing value has a new type. (There are other legal conversions, such as from integer to floating point, that do create a new value.)

- It's an idiom in Go programs to convert the type of an expression to access a different set of methods. 


<-[code 13](https://play.golang.org/p/a7YDyrrARp)

#### Interface conversions and type assertions

- type switches

<-code 14
```go
type Stringer interface {
    String() string
}

var value interface{} // Value provided by caller.
switch str := value.(type) {
case string:
    return str
case Stringer:
    return str.String()
}
```

- type assertions

<-code 15
```go
str, ok := value.(string)
if ok {
    fmt.Printf("string value is: %q\n", str)
} else {
    fmt.Printf("value is not a string\n")
}
```
 

#### Generality
- [ref](http://blog.jonathanoliver.com/golang-has-generics/)
- If a type exists only to implement an interface and will never have exported methods beyond that interface, there is no need to export the type itself. Exporting just the interface makes it clear the value has no interesting behavior beyond what is described in the interface
 
- [crc32.NewIEEE](https://golang.org/pkg/hash/crc32/#NewIEEE), [adler32.New](https://golang.org/pkg/hash/adler32/#New)

<-code 16
```go
type Block interface {
    BlockSize() int
    Encrypt(src, dst []byte)
    Decrypt(src, dst []byte)
}

type Stream interface {
    XORKeyStream(dst, src []byte)
}

// NewCTR returns a Stream that encrypts/decrypts using the given Block in
// counter mode. The length of iv must be the same as the Block's block size.
func NewCTR(block Block, iv []byte) Stream
```
- NewCTR applies not just to one specific encryption algorithm and data source but to any implementation of the Block interface and any Stream. Because they return interface values, replacing CTR encryption with other encryption modes is a localized change. 

- [ref](https://play.golang.org/p/uFAl2BlhUy)
 

#### Interfaces and methods
- Since almost anything can have methods attached, almost anything can satisfy an interface. 

<-code 16
```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

<-code 17
```go
// Simple counter server.
type Counter struct {
    n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ctr.n++
    fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```

<-code 18
```go
// Simpler counter server.
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    *ctr++
    fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```

<-code 19
```go
// A channel that sends a notification on each visit.
// (Probably want the channel to be buffered.)
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ch <- req
    fmt.Fprint(w, "notification sent")
}
```

<-code 20
```go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
```
- The HandlerFunc type is an adapter to allow the use of ordinary functions as HTTP handlers. If f is a function with the appropriate signature, `HandlerFunc(f)` is a Handler that calls f.

<-code 21
```go
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}

http.Handle("/args", http.HandlerFunc(ArgServer))
```
- We have made an HTTP server from a struct, an integer, a channel, and a function, all because interfaces are just sets of methods, which can be defined for (almost) any type.


