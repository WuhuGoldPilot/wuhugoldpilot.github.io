---
layout: article
title: Effective Go
tags: GoLang
sharing: true
show_author_profile: true
typora-copy-images-to: ..\..\assets\blog_img
typora-root-url: ..\..
key: 2021-07-30-golang-effectivego
comment: true
mermaid: true
chart: true
---

本文翻译自[Effective Go](https://golang.org/doc/effective_go)，语句不太通顺，也可能有某些错误，欢饮指正！

## Go代码规范

### Getter

go不提供自动的getter和setter，你可以自己提供。需要注意的是，如果你有一个字段owner【小写，不导出的】，那么getter方法应该是Owner() 【大写，导出的】而不是GetOwner()，setter方法为常见的SetOwner()。

examples:

```go
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

### Interface Names

单一方法接口由方法名称加上-er后缀或类似的修饰来构造名词：`Reader`,`Writer`,`Formatter`,`CloseNotifier`.

### MixedCaps

go中习惯使用`MixedCaps`和`mixedCaps`来写多字的复合名。

### Redeclaration and reassignment

```go
f, err := os.Open(name)
d, err := f.Stat()
```

对于err来说，虽然在第二行出现了短变量声明，但是实际上只是一个重赋值，而不是重新声明。`f.Stat`使用的是上面声明的err，并且重新赋值。

### Switch

switch的表达式不需要为常量或者整数，case从上到下计算，直至找到匹配，如果switch没有表达式，则默认为`trrue`。因此这很常用将`if-else`改写为`switch`。

```go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

go的switch没有自动降级，但是`cases`可以用`,`分割的列表表示。

```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```

尽管不像其他类C语言一样常见使用`break`，但是`break`也是可以停止一个`switch`。但是，有时候需要打断周围的循环，而不是`switch`。在go中，可以在循环上放置一个标签并`breaking`该标签来实现。example：

```go
Loop:
	for n := 0; n < len(src); n += size {
		switch {
		case src[n] < sizeOne:
			if validateOnly {
				break
			}
			size = 1
			update(src[n])

		case src[n] < sizeTwo:
			if n+1 >= len(src) {
				err = errShortInput
				break Loop
			}
			if validateOnly {
				break
			}
			size = 2
			update(src[n] + src[n+1]<<shift)
		}
	}
```

当然，`continue`也可以作为可选的标签，但是它只用于循环。

### Type  Switch

`switch`也可以用于发现接口变量的动态类型。这种*type switch*使用类型断言的语法，关键字`type`在括号里面。如果`switch`在表达式中声明了一个变量，则该变量在每个子句中都有对应的类型。在这种情况下重用名称是惯用的，实际上每种情况下都声明了一个具有相同名称但类型不同的新变量，

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```

### Named result parameters

go函数的返回结果可以命名并作为常规变量，像传入参数一样。当被命名时，它们在函数开始时被初始化为它们的类型的零值，如果函数执行时不带参数的retrun语句，则结果参数的当前值将用作返回值。

命名不是强制性的，但是它们可以使代码更短更清晰，它们是文档，如果我们命名`nextInt`的结果，我们可以知道返回的`int`是哪个。

```go
func nextInt(b []byte, pos int) (value, nextPos int) {
```

因为命名结果已初始化并绑定到未修饰的返回值，它们可以简单和清晰。example：

```go
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

### Defer

go的`defer`语句会在`return`之前执行`defer`的函数的回调，这是一种不寻常但是有效的方法去处理诸如资源释放的情况，无论函数在哪条路径返回。example：

```go
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

定义一个`defer`回调函数有两个好处：一是它保证不会让你忘记关闭文件，如果你修改你的方法增加一条新的返回路径，你很容易忘记这件事；二是这意味着你需要将它定义在`open`旁边，这比定义在函数结尾更加清晰。

延迟函数的参数（如果函数是方法，则包含接收者）在延迟执行时计算，而不是在调用执行时计算。除了避免担心在执行时变量会改变值，这意味着单个延迟调用可以延迟多个函数执行。延迟函数以`LIFO`的顺序执行，example：

```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

输出：

```go
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

对于习惯其他语言的块级管理资源的程序员来说，`defer`可能看起来很奇怪，但它最强大和最有趣的应用程序恰恰来自它不是基于块的而是基于函数的事实。

### Data

### Allocation with `new` 

go有两个分配内存的原语：`new`和`make`，它们做不同的事且适用不同的对象。

`new`是一个分配内存的内置函数，但与其他语言的同名函数不同，它不会初始化内存，只会将其归零。因此`new(T)`为类型为`T`的新项分配零存储并返回其地址，类型为`*T`的值。在go术语中，它返回一个指向新分配的`T`类型零值的指针。

由于`new`返回的内存已归零，因此在设计数据结构时安排每种类型的零值无需进一步初始化即可使用是有帮助的。这意味着数据结构的用户可以使用`new`创建一个并立即可以使用它。

零值有用的属性是传递性的，example：

```go
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```

`SyncedBuffer`类型的值可以在分配或者声明后直接使用。在下一个片段中，`p`和`v`无需进一步安排即可正常工作。

```go
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

### Allocation with `make`

内置函数`make(T,args)`的用途不同于`new(T)`，它只创建切片，映射和通道，它返回一个类型为`T`（不是`*T`）

的初始化（未清零）值。区分的原因是这三种类型在幕后在使用前必须初始化的数据结构引用

。例如切片是一个三项描述符，包含指向数据（在数组内）、长度和容量的指针，并且在这些项被初始化之前，切片为`nil`。对于切片、映射和通道，`make`会初始化内部数据结构并准备要使用的值，例如：

```go
make([]int, 10, 100)
```

分配一个包含100个整数的数组，然后创建一个长度为10、容量为100的切片结构，指向数组的前10个元素。相比之下，`new([]int))`返回一个指向新分配的、归零的切片结构的指针，即指向`nil`切片值的指针。下面是关于`new`和`make`的不同之处的例子：

```go
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

// Unnecessarily complex:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Idiomatic:
v := make([]int, 100)
```

记得`make`只用于映射、切片、和通道并且不返回指针，使用`new`获得显式指针分配或显式获取变量的地址。

### Arrays

数组在规划内存的详细布局时很有用，有时可以帮助避免分配，但主要是切片的构造块。数组在C和go中有很大的不同，在go中：

- 数组是值，将一个数组分配给另一个会复制所有的值；

- 特别是，如果你将一个数组传给一个函数，他会收到一个数组的副本，而不是一个指向它的指针；

- 数组的大小是其类型的一部分，`[10]int`和`[20]int`的类型是不同的。

  

`value`属性可以很有用也可以很昂贵，如果你想要类似C的行为和效率，你可以传递一个指向数组的指针：

```go
func Sum(a *[3]float64) (sum float64) {
    for _, v := range *a {
        sum += v
    }
    return
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)  // Note the explicit address-of operator
```

但即使是这种风格也不是go惯用的，改用切片更加好。

### Slices 

切片包装数组，为数据序列化提供更通用、更强大、更方便的接口。除了具有显式维度的项（例如转换矩阵），go中的大多数数组编程都是使用切片而不是简单数组完成的。

切片保存对底层数组的引用，如果将一个切片分配给另一个切片，则它们都引用同一个数组。如果一个函数接受一个切片参数，它对切片元素所做的更改将对调用者可见，类似于传递一个指向底层数组的指针。因此，`Read`函数可以接受切片参数而不是指针和计数，切片内的长度设置了要读取的数据量的上限。下面是在`os`包的`File`类型的`Read`方法的签名：

```go
func (f *File) Read(buf []byte) (n int, err error)
```

该方法返回读取的字节数和错误值（如果有），要读入较大缓冲区的前32个字节，请对缓冲区进行切片：

```go
n, err := f.Read(buf[0:32])
```

这种切片很常见也很高效。切片的长度可以改变，只要它仍然适合底层数组的限制。只需将其分配给自身的一部分，切片的容量，可通过内置函数`cap`访问，报告切片可能采用的最大长度。下面是一个将数据附加到切片的函数，如果数据超出容量，则重新分配切片，返回结果切片。该函数使用`len`和`cap`在应用于`nil`切片时合法的事实，并返回0.

```go
func Append(slice, data []byte) []byte {
    l := len(slice)
    if l + len(data) > cap(slice) {  // reallocate
        // Allocate double what's needed, for future growth.
        newSlice := make([]byte, (l+len(data))*2)
        // The copy function is predeclared and works for any slice type.
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:l+len(data)]
    copy(slice[l:], data)
    return slice
}

```

之后我们必须返回切片，因为虽然`Append`可以修改切片的元素，但是切片本身（保存指针、长度和容量的运行时数据结构）是按值传递的。附加数据都切片中是非常有用的，它被`append`内置函数捕获。

### Two-dimensional slices

go的数组和切片是一维的，要创建二维数组或切片的等效项，必须定义一个数组数组或切片数组，如下所示：

```go
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.	
```

因为切片是可变长度的，所以可以让每个内部切片的长度不同。这是一个常见的情况，如下例子所示，每行都有一个独立的长度：

```go
text := LinesOfText{
	[]byte("Now is the time"),
	[]byte("for all good gophers"),
	[]byte("to bring some fun to the party."),
}
```

有时候分配一个2D切片是有必要的，有两个方法可以实现它。一种是独立分配每个切片；另一种是分配单个数组并将各个切片指向其中。使用哪种方式取决于你的应用，如果切片可能会增长或者缩小，则应单独分配以避免覆盖下一行；如果没有，使用单个分配构造对象会更有效。下面是两个分配方式的例子：

一次一行：

```go
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
```

一个分配，分成几行：

```go
// Allocate the top-level slice, the same as before.
picture := make([][]uint8, YSize) // One row per unit of y.
// Allocate one large slice to hold all the pixels.
pixels := make([]uint8, XSize*YSize) // Has type []uint8 even though picture is [][]uint8.
// Loop over the rows, slicing each row from the front of the remaining pixels slice.
for i := range picture {
	picture[i], pixels = pixels[:XSize], pixels[XSize:]
}
```

### Maps

映射是一种方便且强大的内置数据结构，它将一种类型（键）的值与另一种类型（元素或值）的值相关联。
键可以是定义了相等运算符的任何类型，例如整数、浮点数和复数、字符串、指针、接口（只要动态类型支持相等）、结构和数组。切片不能用作映射键，因为它们没有定义相等性。像切片一样，映射保存对底层数据结构的引用。如果将`map`传递给更改`map`内容的函数，则更改将在调用方中可见。
可以使用带有冒号分隔的键值对的常用复合文字语法构建映射，因此在初始化期间很容易构建它们。

```go
var timeZone = map[string]int{
    "UTC":  0*60*60,
    "EST": -5*60*60,
    "CST": -6*60*60,
    "MST": -7*60*60,
    "PST": -8*60*60,
}
```

分配和获取映射值在语法上看起来就像对数组和切片执行相同的操作，只是索引不需要是整数。

```go
offset := timeZone["EST"]
```

尝试使用映射中不存在的键获取映射值将返回映射中类型的零值。例如，如果映射包含整数，查找不存在的键将返回 0。集合可以实现为值类型为 bool 的映射。将映射条目设置为 true 以将值放入集合中，然后通过简单的索引对其进行测试。

```go
attended := map[string]bool{
    "Ann": true,
    "Joe": true,
    ...
}

if attended[person] { // will be false if person is not in the map
    fmt.Println(person, "was at the meeting")
}
```

有时您需要将缺失的值与零值区分开来。是否有“UTC”的条目或者是 0，因为它根本不在`map`中？你可以用多重赋值的形式来区分。

```go
var seconds int
var ok bool
seconds, ok = timeZone[tz]
```

出于显而易见的原因，这被称为“逗号确定”习语。在这个例子中，如果 tz 存在，秒将被适当地设置并且 ok 为真；如果没有，秒将被设置为零，确定将是假的。这是一个将它与一个很好的错误报告放在一起的函数：

```go
func offset(tz string) int {
    if seconds, ok := timeZone[tz]; ok {
        return seconds
    }
    log.Println("unknown time zone:", tz)
    return 0
}
```

要在不担心实际值的情况下测试`map`中是否存在，您可以使用空白标识符 (_) 代替该值的常用变量。

```go
_, present := timeZone[tz]
```

要删除`map`的值，请使用 delete 内置函数，其参数是要删除的`map`和键。

```go
delete(timeZone, "PDT")  // Now on Standard Time
```

### Printing 

go的格式化打印和c的`printf`类似但是比它更丰富和更普遍。这些方法放在`fmt`包中，并且有大写字母的名字：`fmt.Printf`, `fmt.Fprintf`, `fmt.Sprintf`等。字符串函数（`Sprintf`等）返回一个字符串而不是填充提供的缓冲区。
你不需要提供格式化字符串，对于`Printf`, `Fprintf`和`Sprintf`中的每一个，都有另一对函数，例如`Print`和`Println`。这些函数不采用格式化字符串，而是为每个参数生产默认格式。`Println`函数在每个参数之前插入一个空格并在输出中附加一个换行符，而`Print`仅当两边参数都是字符串时才添加空格。下面这个例子，每一行的输出都是一样的：

```go
fmt.Printf("Hello %d\n", 23)
fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
fmt.Println("Hello", 23)
fmt.Println(fmt.Sprint("Hello ", 23))
```

格式化打印函数`fmt.Fprintf`和他们的同类将任何实现`io.Writer`接口的对象作为第一个参数，变量`os.Stout`和`os.Stderr`是熟悉的实例。
下面是不同于C的部分。首先，诸如%d之类的数字格式不采用符号或大小的标志；其次，打印线程使用参数的类型来决定这些属性。

```go
var x uint64 = 1<<64 - 1
fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))
```

打印：

```go
18446744073709551615 ffffffffffffffff; -1 -1
```

如果你只想要默认的转换，比如整数的十进制，你可以使用笼统的格式`%v`（value），结果正是`Print`和`Println`所产生的结果。此外，此格式可以打印任何值，包括数组，切片，结构和映射。这是上一节定义时区地图的打印语句：

```go
fmt.Printf("%v\n", timeZone)  // or just fmt.Println(timeZone)
```

输出：

```go
map[CST:-21600 EST:-18000 MST:-25200 PST:-28800 UTC:0]
```

对于映射，`Printf`和它相似的方法按字典排序对输出进行排序。
打印结构体时，修改后的格式`%+v`用它们的名称注释结构体的字段，对于任何值，替代格式`%#v`以完整的go语法打印值。

```go
type T struct {
    a int
    b float64
    c string
}
t := &T{ 7, -2.35, "abc\tdef" }
fmt.Printf("%v\n", t)
fmt.Printf("%+v\n", t)
fmt.Printf("%#v\n", t)
fmt.Printf("%#v\n", timeZone)
```

打印：

```go
&{7 -2.35 abc   def}
&{a:7 b:-2.35 c:abc     def}
&main.T{a:7, b:-2.35, c:"abc\tdef"}
map[string]int{"CST":-21600, "EST":-18000, "MST":-25200, "PST":-28800, "UTC":0}
```

(注意&符号)当应用于`string`或`[]byte`类型的值时，引用的字符串格式也可以通过`%q`获得。如果可能，替代格式`%#q`将使用反引号（`q%`格式也适用于整数和`rune`，生成单引号`rune`常量）。此外。`%x`适用于字符串，字节数组和字节切片以及整数，生成一个长的十六进制字符串，也可以使用`% x`格式在每个字节之中放置空格。

另一种便利的格式`%T，它打印值的类型。

```go
fmt.Printf("%T\n", timeZone)
```

打印：

```go
map[string]int
```

如果你想要控制自定义类型的默认格式，只需要在类型上定义一个带有签名`String() string`的方法。对于简单类型`T`，可能如下：

```go
func (t *T) String() string {
    return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
}
fmt.Printf("%v\n", t)
```

打印：

```go
7/-2.35/"abc\tdef"
```

如果你需要打印`T`类型的值以及指向`T`的指针，则`String`的接收者必须是值类型，这个例子使用了一个指针，因为它对于结构类型更加有效和惯用。
我们的`String`方法可以调用`Sprintf`是因为打印线程是完全可重入的并且可以以这种方式包装。然而，关于这种方法有一个重要的细节需要理解：不要通过调用`Sprintf`会循环调用你的`String`方法去构造一个`String`方法。如果`Sprintf`调用尝试将接收器直接打印为字符串，则可能发生这种情况，而后者又会再次调用该方法。如下例所示：这是一个常见且容易犯的错误：

```go
type MyString string

func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.
}
```

修复也很容易：将参数转换为基本字符串类型，它没有方法调用。

```go
type MyString string
func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
}
```

```go
func Printf(format string, v ...interface{}) (n int, err error) {}
```

在函数`Printf`中，v的作用类似于`[]inteface{}`类型的变量，但如果将其传递给另一个可变参数函数，它的作用类似于常规参数列表。下面的例子是我们使用上面的函数实现的，它将参数直接传递给`fmt.Sprintln`以实现格式化：

```go
// Println prints to the standard logger in the manner of fmt.Println.
func Println(v ...interface{}) {
    std.Output(2, fmt.Sprintln(v...))  // Output takes parameters (int, string)
}
```

我们在对`Sprintln`的嵌套调用中在v之后写入`...`告诉编译器将v视为参数列表，否则它只会将v作为单个切片参数传递。

`...`参数也可以指定类型，例如`...int`用于选择整数列表中最小者的`min`函数：

```go
func Min(a ...int) int {
    min := int(^uint(0) >> 1)  // largest int
    for _, i := range a {
        if i < min {
            min = i
        }
    }
    return min
}
```

### Append

`append`的签名与我们上面自定义的`Append`函数不同：

```go
func append(slice []T, elements ...T) []T
```

其中`T`是任何给定类型的占位符，你实际上无法在go中编写类型`T`由调用者确定的函数。这就是`append`内置的原因：它需要编译器的支持。
`append`的作用就是将元素追加到切片的末尾并返回结果，结果需要返回是因为与我们手写的`Append`一样，底层数组可能会改变。一个简单的例子：

```go
x := []int{1,2,3}
x = append(x, 4, 5, 6)
fmt.Println(x)
```

打印`[1 2 3 4 5 6]`。所以`append`和`Printf`一样，接受任意数量的参数。
但是我们想要的是添加一个切片到另一个切片，这使用`...`很容易达到。

```go
x := []int{1,2,3}
y := []int{4,5,6}
x = append(x, y...)
fmt.Println(x)
```

如果没有`...`，编译器会报错：`y`不是类型`int`。

### Initialization 初始化

尽管和c和c++的初始化没什么大的不同，但是go的初始化要强大的多。复杂结构的组建、初始化对象中的顺序问题、甚至在不同的包，go都能正确地处理。

### Constants 常数

go的常熟是`constant`，它们在编译时期就已经创建，即使它们被定义为函数的本地变量，它们只可以是数字、字符(rune)、字符串和布尔值。因为编译时期的限制，定义它们只能使用编译器能识别的常量表达式。例如：`1<<3`是一个常量表达式，`math.Sin(math.Pi/4)`就不是，因为函数`math.Sin`回调是在运行时期。
在go中，列举常量使用`iota`枚举器。因为`iota`是表达式的一部分且可以隐式重复，所以这很容易定义一组复杂的常量值。

```go
type ByteSize float64

const (
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```

将诸如`String`之类的方法附加到任何用户定义的类型使得任意值可以自动格式化以进行打印，虽然你看到它经常被应用到结构体，但是这种技术对于标量也很有用，例如像`ByteSize`这种浮点类型。

```go
func (b ByteSize) String() string {
    switch {
    case b >= YB:
        return fmt.Sprintf("%.2fYB", b/YB)
    case b >= ZB:
        return fmt.Sprintf("%.2fZB", b/ZB)
    case b >= EB:
        return fmt.Sprintf("%.2fEB", b/EB)
    case b >= PB:
        return fmt.Sprintf("%.2fPB", b/PB)
    case b >= TB:
        return fmt.Sprintf("%.2fTB", b/TB)
    case b >= GB:
        return fmt.Sprintf("%.2fGB", b/GB)
    case b >= MB:
        return fmt.Sprintf("%.2fMB", b/MB)
    case b >= KB:
        return fmt.Sprintf("%.2fKB", b/KB)
    }
    return fmt.Sprintf("%.2fB", b)
}
```
表达式`YB`打印出`1.00YB`，而`ByteSize(1e13)`打印`9.09TB`。
这里使用`Sprintf`实现`ByteSize`的`String`方法是安全的（避免了循环调用）不是因为一次转换而是因为它使用`%f`调用`Sprintf`，这不是一个字符串格式：`Sprintf`只会当它是一个`string`的时候调用它的`String`方法，`%f`拿到的是一个浮点数。

### Variables 变量

变量可以像常量那样初始化，但是初始化程序可以是在运行时计算的一般表达式。

```go
var (
    home   = os.Getenv("HOME")
    user   = os.Getenv("USER")
    gopath = os.Getenv("GOPATH")
)
```

### The init function 初始化函数

最后，每一个源文件都可以定义自己的`init`函数去设置需要的任何变量（实际上每个文件可以有多个`init`函数），最后的意思是：`init`函数在所有定义在包中的变量初始化后调用，这些变量会在所有引入的包初始化后初始化。
除了不能表示为声明的初始化之外，`init`函数也常用于正式执行之前的验证和修复程序状态。

```go
func init() {
    if user == "" {
        log.Fatal("$USER not set")
    }
    if home == "" {
        home = "/home/" + user
    }
    if gopath == "" {
        gopath = home + "/go"
    }
    // gopath may be overridden by --gopath flag on command line.
    flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

## Method 方法

### Pointers vs. Values 指针和值的对比

就像我们在`ByteSize`中看到的那样，方法可以为任何命名类型定义（除了一个指针或者一个接口），接收者不必为一个结构体。  

在上面的切片讨论中，我们写了一个`Append`函数，我们可以将其定义为切片上的方法。为了达成这样，我们首先定义一个命名变量可以去绑定方法，然后将它作为方法的接收者。

```go
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
    // Body exactly the same as the Append function defined above.
}
```

这仍然需要方法返回一个更新过的切片，我们可以使用一个`ByteSilce`指针作为接收者去防止重定义方法，所以
这个方法可以重写它的调用切片。

```go
func (p *ByteSlice) Append(data []byte) {
    slice := *p
    // Body as above, without the return.
    *p = slice
}
```

实际上我们可以做的更好，如果我们修改我们的函数让它看起来像标准的`Write`方法一样：

```go
func (p *ByteSlice) Write(data []byte) (n int, err error) {
    slice := *p
    // Again as above.
    *p = slice
    return len(data), nil
}
```

然后`*ByteSlice`类型满足接口`io.Writer`的标准，这非常方便。例如：我们可以打印一个：

```go
    var b ByteSlice
    fmt.Fprintf(&b, "This hour has %d days\n", 7)
```

我们传一个`ByteSlice`的指针是因为只有`*ByteSlice`满足`io.Writer`。关于指针和值作为接收者的规则是值方法能被指针和值调用，而指针方法只能被指针调用。

有这条规则是因为指针方法能修改接收者，用一个值调用它们会返回一个复制的值，所以任何修改都会被丢弃，所以从语言上不允许这种错误，因此这是一个方便的异常。当值可寻址时，语言会通过自动插入地址运算符来处理对值调用指针方法的常见情况。在我们的例子中，变量`b`是可寻址的，所以我们可以使用`b。Write`调用它的`Write`方法，编译器会为我们自动重写为`(&b).Write`。

顺便一提，在字节切片上使用`Write`是实现`byte.Buffer`的核心。

## Interfaces and other types 接口和其他类型

## Interfaces 接口

go的接口提供了指定对象的行为的一种方法，如果某些东西能这样做，拿它就能用在这种地方。我们已经看过几个简单2的例子：定制化打印能够通过`String`方法实现，而`Fprintf`可以通过任何实现了`Write`方法生成输出。接口只包含一到两个方法在go中是很常见的，并且它经常被命名为方法的衍生出的名字，例如`io.Write`就是实现了`Write`方法的接口。

一种类型能实现多个接口，例如一个集合如果实现了`sort.Interface`接口，它就能通过包`sort`中的例程对集合进行排序。这个集合包含了`Len()`, `Less(i, j int) bool` 和 `Swap`方法，并且它能有定制化的格式化方法，在这个人为的例子中，`Sequence`都满足这两部分：

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

// Copy returns a copy of the Sequence.
func (s Sequence) Copy() Sequence {
    copy := make(Sequence, 0, len(s))
    return append(copy, s...)
}

// Method for printing - sorts the elements before printing.
func (s Sequence) String() string {
    s = s.Copy() // Make a copy; don't overwrite argument.
    sort.Sort(s)
    str := "["
    for i, elem := range s { // Loop is O(N²); will fix that in next example.
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}
```

### Conversions 转换

`Sequence`的`String`方法重复了`Sprint`为切片做的工作，（它的复杂度为`O(N²)`，这并不算很高）我们可以尝试在调用`Sprint`之前把`Sequence`转换为`[]int`，这样可以分享工作并加快速度。

```go
func (s Sequence) String() string {
    s = s.Copy()
    sort.Sort(s)
    return fmt.Sprint([]int(s))
}
```

这个方法是另外的一个转换技术的例子安全地通过调用`String`方法调用`Sprint`。因为如果我们忽略类型名字`Sequence`和`[]int`两种类型是相同的，它们互转是合法的。转换不创建一种新类型，它只是暂时表现为现在值具有新类型。（另外一些合法的转换，例如整形转浮点型，这就会真的创建一个新的值）

转换一个表达式的类型去方法问不同的方法集是一种go的惯用语法。例如：我们可以使用现有的类型`sort.IntSlice`将整个例子简化：

```go
type Sequence []int

// Method for printing - sorts the elements before printing
func (s Sequence) String() string {
    s = s.Copy()
    sort.IntSlice(s).Sort()
    return fmt.Sprint([]int(s))
}
```

与其让`Sequence`实现多个接口（soring和printing），我们使用数据能够转换为多个类型，每种类型都能完成一种工作去代替。这在实践中不寻常，但是却很有效。

### Interface conversions and type assertions 接口转换和类型断言

`Type switch`是类型转换的一种形式，它们获取一个接口，对于`switch`的每个case，在某种意义上将它转换为case的那种类型。
下面是一个在`fmt.Printf`下的代码怎样使用类型开关转换到`string`类型的简单的例子。如果它已经是一个字符串，我们希望得到接口保存的实际值，而如果它有一个`String`方法，我们想要调用该方法的结果。

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

第一个情况找到了具体的值；第二个转换接口到另外的接口，以这种方式混合类型非常好。
我们只关心一种类型怎么办？如果我们知道一个值包含一个字符串并且我们想要提取它？单例类型开关可以，但类型断言也可以。类型断言采用接口值并从中提取指定显式类型的值。语法借用了打开类型开关的子句，但是使用了显示类型而不是`type`关键字：

```go
value.(typeName)
```

结果是一个具有静态类型`typeName`的新值，该类型必须为接口持有的具体类型，或者是值可以转换为的第二个接口类型。要提取在值中的字符串，我们可以这样写：

```go
str := value.(string)
```

但是如果证明该值不包含字符串，程序会因运行时错误而崩溃。为了防止出现这种情况，使用`"comma.ok"`习语来安全地测试该值是否为字符串：

```go
str, ok := value.(string)
if ok {
    fmt.Printf("string value is: %q\n", str)
} else {
    fmt.Printf("value is not a string\n")
}
```

如果类型断言失败，`str`仍然存在并且是字符串类型，但是它会是一个零值：一个空字符串。作为能力的说明，这是一个`if-else`语句，相当于打开此部分的类型开关。

```go
if str, ok := value.(string); ok {
    return str
} else if str, ok := value.(Stringer); ok {
    return str.String()
}
```

### Generality 概论

如果一个类型只是为了实现一个接口并且永远不会暴露方法到接口之外，这样不需要导出类型本身。仅导出接口可以清楚地表明该值除了接口中描述的之外没有其他有趣的行为。它还避免了对通用方法的每个实例重复文档的需要。

在这些案例中，构造函数应该返回一个接口而不是实现类型。例如，在哈希库中，`crc32.NewIEEE, adler32.New`都返回接口类型`hash.Hash32`。在go中用`Adler-32`替代`CRC-32`算法只需要改变构造函数的调用，其余代码不受算法改变的影响。

### Interfaces and methods 接口和方法

由于任何东西都可以附加方法，所以几乎所有东西都能满足接口。一个说明性例子就是在定义了`Header`接口的`http`包中，任何实现了`Header`接口的对象都能为`HTTP`提供服务。

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

`ResponseWriter`本身是一个接口，它提供对将返回给客户端所需的方法的访问。这些方法包含标准的`Write`方法，所以一个`http.ResponseWriter`可是在任何`io.Writer`可以被使用的地方使用。`Request`是一个包含来自客户端请求的解析表示的结构体。

为了简洁，我们忽略post请求，认为所有的http请求都是get，这种简化不会影响处理程序的设置方式。这是一个处理程序的简单实现，用于计算页面被访问的次数。

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

(与我们的主题保持一致，请注意`Fprintf`怎么打印到`http.ResponseWriter`)在真实的服务器中，访问`cte.n`需要防止并发访问，请参阅`sync`和`atomic`包以获取建议。

以供参考，下面是如何将这样的服务器附件到URL树上的节点。

```go
import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)
```

但是为什么把`Counter`定义为结构体？它只需要一个整数。（接收者需要一个指针，以便调用者可以看到增量）

```go
// Simpler counter server.
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    *ctr++
    fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```

如果你的程序有一些需要通知页面已被访问的内部状态怎么办？将`channel`绑定到web页面。

```go
// A channel that sends a notification on each visit.
// (Probably want the channel to be buffered.)
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ch <- req
    fmt.Fprint(w, "notification sent")
}
```

最后，假设我们想要在`/arg`上显示调用服务器二进制文件时使用的参数。写一个函数打印这些参数很容易：

```go
func ArgServer() {
    fmt.Println(os.Args)
}
```

我们怎么将其转换为http服务器？我们可以使`ArgServer`成为某种类型的方法，我们忽略其值。但有一个更简洁的方法。因为我们可以为除指针和接口之外的任何类型定义一个方法。我们可以为一个函数编写一个方法，`http`包包含一下代码：

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

`HandleFunc`是一种带有方法的类型，`ServerHTTP`因此该类型可以为HTTP请求提供服务。看一下方法的实现：接收者是一个函数`f`，并且该方法调用`f`。这可能看起来很奇怪，但它与接收者是一个通道以及在通道上发送的方法并没有什么不同。

为了让`ArgServer`成为一个HTTP服务器，我们首先需要将它修改成正确的签名。

```go
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}
```

现在`ArgServer`和`Handlefunc`有相同的签名，所以它能够转换成这种相同去访问它的方法，就像我们转换`Sequence`到`IntSlice`去访问`IntSlice.Sort`一样，设置它的代码很简洁：

```go
http.Handle("/args", http.HandlerFunc(ArgServer))
```

当用户访问`/arg`页面时，安装在该页面上的处理程序具有值`ArgServer`和类型`HandleFunc`。HTTP服务器将调用该类型的`ServerHTTP`方法，将`ArgServer`作为接收方，然后调用`ArgServer`(通过`HandleFunc.ServerHTTP`中的`f(w,req)`调用)。然后将显示参数。

在这一章节我们通过一个结构体、一个整数、一个通道和一个函数创建了一个HTTP服务器，这都是因为接口只是一个方法的集合，它几乎可以为任何类型定义。

### The blank identifier 空白标识符

我们已经多次提到了空白标识符了，在`for range`循环和`maps`章节。空白标识符可以被分配和声明为任何值和任何类型，并无害地丢弃该值。这和写Unix的`/dev/null`文件有点像：它表示一个只写值，用作占位符，其中需要变量但实际值无关紧要。它的用途超出了我们已经见过的用途。

### The blank identifier in multiple assignment 多重赋值中的标识符

在`for range`循环中使用多重赋值标识符是一般情况的特例。

如果赋值需要多个值在左边，但是其中一个值并不会在程序中用到，在赋值左边的空白标识符防止创建一个虚拟变量并且表明该值被丢弃。例如：当调用一个函数返回一个值和错误，但是仅仅错误是重要的，我们可以使用空白标识符表明丢弃无关重要的值。

```go
if _, err := os.Stat(path); os.IsNotExist(err) {
	fmt.Printf("%s does not exist\n", path)
}
```

偶尔你会看见一些代码丢弃了错误值去忽略错误，这是非常糟糕的。总是检查错误返回，提供它们是有原因的。

```go
// Bad! This code will crash if path does not exist.
fi, _ := os.Stat(path)
if fi.IsDir() {
    fmt.Printf("%s is a directory\n", path)
}
```

### Unused imports and variables 未使用的导入和变量

导入一个包或声明一个变量不使用是错误的。未使用的导入会使程序变得臃肿和编译缓慢，而初始化未使用的变量至少是计算的浪费和可能表明存在更大的错误。然而，当程序处于活跃开发状态时，经常会出现未使用的导入和变量，删除它们只是为了编译继续进行，以后再次需要它们的时候就会觉得很烦人。空白标识符为这种尝试提供了解决方法。

这个写道一半的程序有两个未使用的导入(`fmt, io`)和一个未使用的变量(`fd`)，所以它会编译错误，但是很高兴看见代码到目前为止是正确的。

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
}
```

要消除对未使用的导入的抱怨，请使用空白标识符来引入导入包的符号。类似地，分配`fd`一个空白标识符可以消除未使用变量的报错，下面的程序版本可以编译：

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done.
var _ io.Reader    // For debugging; delete when done.

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}
```

按照惯例，用于消除导入错误的全局声明应该在导入之后立即出现并进行注释，以便于找到它们并提醒之后清理。

### Import for side effect 导入副作用

之前例子的`fmt`或者`io`未使用的导入最终会被用掉或者移除：空白声明将代码标识未正在进行的工作。但有时导入一个包只是为哦了它的副作用而没有明确的使用。例如：在`init`函数期间，`net/http/pprof`包注册提供调试信息的HTTP处理程序。它有导出的API，但是大多数客户端只需要处理器注册和通过web页面访问数据。为了导入包仅仅为了它们的副作用，可以用空白标识符重命名包。

```go
import _ "net/http/pprof"
```

这种导入形式表明包导入只是为了它的副作用，因为除此之外没有任何地方用到这个包：在这个文件，它没有名字。（如果它用了，并且我们不使用它的名字，编译器会报错）

### Interface checks 接口检查

就如我们看见的在上面的关于接口的讨论，实现一个接口的类型不需要明确地声明。相反地，一个类型实现接口只需要实现接口的方法即可。在实际中，大多数接口转换是静态的因此检查接口是在编译时期。例如：传递一个`*os.File`到一个需要一个`io.Reader`参数的函数会编译不通过，除非`*os.File`实现了`io.Reader`接口。

尽管也有一些接口检查在运行时发生。例如：定义了`Marshaler`接口的`encoding/json`包就是一个实例。当JSON编码器收到一个实现了该接口的值时，编码器会调用值的编码方法去转换它成一个JSON，而不是使用标准转换方法。编码器在运行时期像这样使用类型断言检查这个属性：

```go
m, ok := val.(json.Marshaler)
```

如果只是需要知道是否一个类型实现了一个接口，而不需要使用这个接口，例如只是检查它的错误，使用空白标识符去忽略类型断言的值：

```go
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
```

出现这种情况的一个地方时当需要在实现类型的包中保证它实际满足接口时。例如：如果一个类型`json.RawMessage`需要一个自定义的JSON表示，它需要实现`json.Marshaler`接口，但是没有让编译器自动验证的静态转换这一点。如果类型无意中无法满足接口，JSON编码器仍然可以工作，但是它不会使用自定义实现。为了保证实现是正确的，使用空白标识符的全局性声明可以在包中使用：

```go
var _ json.Marshaler = (*RawMessage)(nil)
```

在这个声明中，涉及将`*RawMessage`转换未`Marshaler`的赋值要求`*RawMessage`实现`Marshaler`，并且将在编译时检查属性。如果`json.Marshaler`接口发生变化，这个包将不能再编译，我们注意到它需要更新。

此构造中出现空白标识符表明该声明仅用于类型检查，而不是创建变量。不要为每种满足接口的类型都这样做，因为按照习俗，这种声明只用于代码中不存在静态转换时，这是非常罕见的。

## Embedding 嵌入

go不提供典型的、类型驱动的子类化概念，但它确实有能力通过在结构或接口中嵌入类型来借用实现的片段。

类型嵌入非常简单，我们已经在`io.Reader`和`io.Writer`接口时提过了。这是它们的定义：

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

`io`包还导出了其他几个接口，这些接口制定了可以实现多个此类方法的对象。例如：同时包含`Read`和`Write`方法的`io.ReadWriter`接口。我们可以通过列出这两个方法明确地指定`io.ReadWriter`，但是嵌入两个接口形成新的接口更容易，更让人回味：

```go
// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
    Reader
    Writer
}
```

就如它所说的一样：一个`ReaderWriter`能够做一个`Reader`和一个`Writer`能做的事。这是一个嵌入接口的联合，只有接口能够嵌入到接口中。

这个基本理念可以应用到结构体，但是有更深远的影响。`bufio`包有两种结构体类型：`bufio.Reader`和`bufio.Writer`，每种都实现了`io`包的类似接口。而且`bufio`也实现了一个缓冲的读写器，它通过使用嵌入将读取器和写入器组合到一个结构中来实现：它列出类型在结构体中但是没有给字段命名。

```go
// ReadWriter stores pointers to a Reader and a Writer.
// It implements io.ReadWriter.
type ReadWriter struct {
    *Reader  // *bufio.Reader
    *Writer  // *bufio.Writer
}
```

嵌合元素是指向结构体的指针，当然在使用之前必须初始化为指向有效的结构。`ReadWriter`结构体可以被写出这样：

```go
type ReadWriter struct {
    reader *Reader
    writer *Writer
}
```

但是为了提升字段的方法并满足`io`接口，我们还需要提供转发方法：例如：

```go
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
    return rw.reader.Read(p)
}
```

通过直接嵌入结构，我们避免了这种薄记。嵌入类型的方法免费提供，这不仅意味着`bufio.ReaderWriter`拥有`bufio.Reader`和`bufio.Writer`的方法，并且它满足`io.Reader`、`io.Writer`和`io.ReadWiter`三个接口。

嵌入和子类化有一个重要的区别。当我们嵌入一个类型时，它的方法会变成外部类型的方法，但是当它们调用时方法的接收者时内部类型而不是外部类型。在我们的例子中：当`bufio.ReadWtiter`的`Read`方法被调用时，它与上面的转发方法写的效果完全一样，接收者是`ReadWriter`的`Reader`字段，而不是`ReadWriter`本身。

嵌入也可以是一种简单的便利。这个例子显示一个嵌入的字段，旁边是一个常规的命名字段。

```go
type Job struct {
    Command string
    *log.Logger
}
```

`Job`类型现在有`Print`, `Printf`, `Println`和其他`*log.Logger`的方法。我们可以给`Logger`一个字段的名字，但是这没必要。现在，一旦初始化，我们就可以使用`Job`记录日志：

```go
job.Println("starting now...")
```

`Logger`是`Job`结构体的常规字段，所以我们可以在`Job`的构造函数中以通常的方法初始化它，像这样：

```go
func NewJob(command string, logger *log.Logger) *Job {
    return &Job{command, logger}
}
```

或者使用复合文字

```go
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
```

如果我们需要直接引用一个嵌入的字段，忽略包限定符的字段的类型名称用作字段名称，就像它在我们的`ReadWriter`结构的`Read`方法中所做的那样。如果我们需要访问`Job`变量`job`的`*log.Logger`，我们可以写`job.Writer`，如果我们想要改进`Logger`的方法会很有用。

```go
func (job *Job) Printf(format string, args ...interface{}) {
    job.Logger.Printf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}
```

嵌入类型引入了名称冲突的问题，但解决它们的规则很简单。首先，字段或方法X在类型的更深层嵌套部分隐藏任何其他项X。如果`log.Logger`包含一个字段或方法`Command`，`job`的`Comand`字段会支配它。其次，如果相同的名称出现在相同的嵌套级别，它通常是一个错误。如果`Job`结构包含一个`Logger`的字段或方法，嵌入一个`log.Logger`是错误的。但是，如果在类型定义之外的程序中从未体积重名，则可以。这个限定提供了一些防止从外部嵌入的类型进行保护的更改。如果添加的字段与另一个子类型中的另一个字段冲突，如果两个字段都没有使用过，则没有问题。

## Concurrency 并发

### Share by communicating 通过交流分享

并发编程是一个很大的话题，这里只有一些go特定的亮点。

由于实现对共享变量的正确访问所需的微妙之处，许多环境的并发编程变得困难。go鼓励一种不同的方法，在这种方法中，共享变量在通道上传递，实际上，从不由单独的执行线程主动共享。在任何给定时间，只有一个`goruntine`可以访问该值。设计上不会发生数据竞争。为了鼓励这种思维方式，我们将其简化为一个口号：

>不要通过共享内存交流，相反地，通过交流共享内存

这种方法可能太过分了，例如，最好通过在整数变量周围放置互斥锁来完成引用计数。但是作为一种高级方法，使用通道去控制访问可以更轻松地写出清晰的、正确的程序。

考虑此模型的一种方法是考虑在一个CPU上运行的典型单线程程序。它不需要同步原语，现在运行另一个实例，它同样不需要同步。现在让它们两个通信，如果通信是同步器，则仍然不需要其他同步。例如，Unix管道就非常适合这个模型。尽管go的并发方法起源于Hoare的`Communicatiing Sequential Processes(CSP)`，它也可以被视为Unix管道的类型安全泛化。

### Goroutines 协程

它们被称为`goroutines`是因为现有的术语-线程、协程、进程等传达了不准确的内涵。一个`goroutine`有一个简单的模型：一个函数和另一个`goruntine`在同一个地址空间中同时执行。它是轻量级的，只比分配堆栈空间多一点消耗。并且堆栈开始时很小，因此它们很便宜，并且根据需要分配（和释放）堆存储来增长。

`goruntines`被多路复用到多个OS线程上，所以如果一个应该阻塞，比如在等待I/O时，其他人继续运行。它们的设计隐藏了很多复杂的线程创建和管理。

使用`go`关键字为函数和方法调用添加前缀，以在新的`goruntine`中运行调用。当调用结束，`goruntine`会静静地退出。（效果类似于在后台运行命令的Unix shell的`&`符号。

```go
go list.Sort()  // run list.Sort concurrently; don't wait for it.
```

函数文字在`goroutine`调用中很方便。

```go
func Announce(message string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println(message)
    }()  // Note the parentheses - must call the function.
}
```

在go中，函数文字是闭包：该实现确保函数引用的变量只要它们处于活动状态就可以存活。

这些示例不太实用，因为这些函数无法发出完成信号。为此，我们需要`channels`

### Channels 通道

像映射一样，`channels`使用`make`分配内存，结果值作为对底层数据结构的引用。如果提供了一个可选的整形参数，它会设置通道的缓冲区大小。对于无缓冲或同步通道，默认值为零。

```go
ci := make(chan int)            // unbuffered channel of integers
cj := make(chan int, 0)         // unbuffered channel of integers
cs := make(chan *os.File, 100)  // buffered channel of pointers to Files
```

无缓冲通道将通信（值的交换）与同步结合，确保两个计算（`goroutine`）处于已知状态。

有很多使用通道的好习语。这是我们开始的一个，在上一节中，我们在后台启动了排序。通道可以允许启动`goroutine`等待排序完成。

```go
c := make(chan int)  // Allocate a channel.
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.
```

接收器总是阻塞直到有数据要接收。如果通道是无缓冲的，发送器会阻塞直至接收方收到该值。如果通道有缓冲区，则发送方只会阻塞，直到值被复制到缓冲区；如果缓冲区已满，这意味着等待某个接收器检索到一个值。

缓冲通道可以像信号量一样使用，例如限制吞吐量。在这个例子中：传入的请求被传递到`handle`，它向通道发送一个值，处理请求，然后从通道接收一个值，为下一个消费者准备“信号量”。通道缓冲区的容量限制了`proecss`同时调用的数量。

```go
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
    sem <- 1    // Wait for active queue to drain.
    process(r)  // May take a long time.
    <-sem       // Done; enable next request to run.
}

func Serve(queue chan *Request) {
    for {
        req := <-queue
        go handle(req)  // Don't wait for handle to finish.
    }
}
```

一旦`MaxOutstanding`个`process`正在执行，任何更多的处理程序都会阻止尝试发送到已填充的通道缓冲区，直到现有处理程序之一完成并从缓冲区接受到值。

但是，这种设计有一个问题：`Server`为每个传入的请求创建一个新的`goroutine`，即使它们中只有`MaxOutstanding`个可以随之运行。因此，如果请求来得太快，程序可能会小号无线资源。我们可以通过改变`Server`来控制`goroutine`的创建来解决这个缺陷。这是一个明显的解决方案，但注意它有一个错误，我们将在随后修复：

```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func() {
            process(req) // Buggy; see explanation below.
            <-sem
        }()
    }
}
```

错误是在go循环中，循环变量在每次迭代中都被重用，因此`req`变量在所有的`goroutine`之间共享。这不是我们想要的，我们需要切薄`req`对每个`goroutine`都是独一无二的。这里有一种方法去实现它：将`req`的值作为参数传递给`goroutine`中的闭包：

```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func(req *Request) {
            process(req)
            <-sem
        }(req)
    }
}
```

将此版本与前一个版本进行比较，以了解闭包声明和运作方式的不同。另一种解决方案是创建一个同名的新变量，如下所示：

```go
func Serve(queue chan *Request) {
    for req := range queue {
        req := req // Create new instance of req for the goroutine.
        sem <- 1
        go func() {
            process(req)
            <-sem
        }()
    }
}
```

写起来可能有点奇怪

```go
req := req`
```

但是在go中是合法和惯用的，你会得到一个同名变量的新版本，故意在本地隐藏循环变量，但每个`goroutine`都是唯一的。

回到编写服务器的一般问题，另一种很好地管理资源的方法是启动固定数量的`handle goroutine`，所有这些`goroutine`都从请求通道读取。`goroutine`的数量限制了同时调用`process`的数量，这个`Server`函数还接受一个将被告知退出的通道，在启动`gorotines`后，它会阻止从该通道接收。

```go
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // Start handlers
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // Wait to be told to exit.
}
```

### Channels of channels 通道的通道

go最重要的特性之一是通道是一流的值，可以像其他任何值一样分配和传递。此属性的一个常见用途是实现安全的并行解复用。

在前一章节的例子中，`handle`是一个理想化的请求处理程序，但我们没有定义它正在处理的类型。如果该类型包含要回复的通道，则每个客户端都可以提供自己的方法答案。这是`request`类型的示意性定义：

```go
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}
```

客户端提供一个函数及其参数，以及请求对象内的一个通道，用于接受应答。

```go
func sum(a []int) (s int) {
    for _, v := range a {
        s += v
    }
    return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// Send request
clientRequests <- request
// Wait for response.
fmt.Printf("answer: %d\n", <-request.resultChan)
```

在服务器端，处理程序函数是唯一改变的东西。

```go
func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```

显然还有很多工作要做才能使它变得现实，但这段代码是一个限速、并行、非阻塞RPC系统框架，而且看不到互斥锁。

### Parallelization 并行

这些想法的另一个应用是跨多个CPU内核并行计算。如果计算可以分解为可以独立运行的单独部分，则可以并行化，并在每个部分完成时发出信号。

假设我们有一个对项目向量执行的昂贵操作，并且这个项目的操作值是独立的，如这个理想化的例子：

```go
type Vector []float64

// Apply the operation to v[i], v[i+1] ... up to v[n-1].
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
    for ; i < n; i++ {
        v[i] += u.Op(v[i])
    }
    c <- 1    // signal that this piece is done
}
```

我们在循环中独立启动这些部分，每个CPU一个。它们可以按任何顺序完成，但这并不重要，我们只是通过在启动所有`goroutine`后排空通道来计算完成信号。

```go
const numCPU = 4 // number of CPU cores

func (v Vector) DoAll(u Vector) {
    c := make(chan int, numCPU)  // Buffering optional but sensible.
    for i := 0; i < numCPU; i++ {
        go v.DoSome(i*len(v)/numCPU, (i+1)*len(v)/numCPU, u, c)
    }
    // Drain the channel.
    for i := 0; i < numCPU; i++ {
        <-c    // wait for one task to complete
    }
    // All done.
}
```

我们可以询问运行时什么值是合适的，而不是`numCPU`创建一个常量值。函数`runtime.NumCPU`返回机器中硬件CPU内核的数量，所以我们可以写：

```go
var numCPU = runtime.NumCPU()
```

还有一个函数`runtime.GOMAXPROCS`，它报告（或设置）用户指定的go程序可以同时运行的内核数。它默认为`runtime.NumCPU`的值，但可以通过设置类似命名的shell环境变量或使用正数调用函数来覆盖。用零调用它只是查询值，因此，如果我们想尊重用户的资源请求，我们应该写：

```go
var numCPU = runtime.GOMAXPROCS(0)
```

一定不要混淆并发（将程序构建为独立执行的组件）和并行性（在多个CPU上并行执行计算以提高效率）的概念。虽然Go的并发特性可以让一些问题易于构建为并行计算，但Go是一种并发语言，而不是并行语言，并不是所有的并行化问题都适合Go的模型。

### A leaky buffer 泄露的缓冲区

并发编程的工具甚至可以让非并发的想法更容易表达。这是一个从RPC包中抽象出来的示例。客户端goroutine循环从某个来源（可能是网络）接收数据。为了避免分配和释放缓冲区，它保留一个空闲列表吗，并使用一个缓冲通道来表示它。如果通道为空，则会分配一个新缓冲区，一旦消息缓冲区准备好，它就会被发送到`serverChan`上的服务器。

```go
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
    for {
        var b *Buffer
        // Grab a buffer if available; allocate if not.
        select {
        case b = <-freeList:
            // Got one; nothing more to do.
        default:
            // None free, so allocate a new one.
            b = new(Buffer)
        }
        load(b)              // Read next message from the net.
        serverChan <- b      // Send to server.
    }
}
```

服务器循环接收来自客户端的每条消息，对其进行处理，并将缓冲区返回到空闲列表。

```go
func server() {
    for {
        b := <-serverChan    // Wait for work.
        process(b)
        // Reuse buffer if there's room.
        select {
        case freeList <- b:
            // Buffer on free list; nothing more to do.
        default:
            // Free list full, just carry on.
        }
    }
}
```

客户端尝试冲`freeList`取回缓冲区，如果没有，它就分配一个新的。服务器发送到`freeList`会将`b`放回空闲列表，除非列表已满，在这种情况下，缓冲区被丢弃在地板上由垃圾收集器回收。(select语句中的default子句在没有其他情况准备好时执行，这意味着select永远不会阻塞。)这个实现只用了几行就构建了一个漏桶空闲列表，依靠缓冲通道和垃圾收集器进行薄记。

## Errors 错误

库例程必须经常向调用者返回某种错误指示。如前所述，Go的多返回值使得在正常返回值旁边返回详细的错误描述变得容易。使用这种特性提供详细的错误信息是一种良好的风格。例如，就如我们看到的一样，`os.Open`在失败时不仅仅返回一个空指针，它还返回了一个描述出现了什么问题的错误值。

按照惯例，错误有类型`error`，一个简单的内置接口。

```go
type error interface {
    Error() string
}
```

库编写者可以在幕后使用使用更丰富的模型自由地实现此接口，从而不仅可以查看错误，还可以提供一些上下文。如前所述，除了通常的`*os.FIle`返回值，`os.Open`还返回一个错误值。如果文件打开成功，则报错为`nil`，但出现问题时，会持有`os.PathError`。

```go
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

`PathErrot`的`Error`生成像这样的字符串：

```text
open /etc/passwx: no such file or directory
```

这样的错误，包括有问题的文件名、操作和它触发的操作系统错误，即使在远离导致它的调用的地方打印也是有用的，它比简单的"no such file or directory"提供更多信息。

在可行的情况下，错误字符串应标识其来源，例如通过前缀命名产生错误的操作或包。例如：在`image`包中，由于未知格式导致的解码错误的字符串表示为:image:unknown format。

关心精确错误细节的调用者可以使用类型开关或者类型断言来查找特定错误并提取细节。对于`PathErrors`，这可能包括检查内部`Err`字段是否存在可恢复的故障。

```go
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        deleteTempFiles()  // Recover some space.
        continue
    }
    return
}
```

这里第二个if语句时另一种类型的断言。如果它失败了，`ok`将会时false，`e`会是`nil`。如果它成功了，`ok`回事true，它意味着错误类型时`*os.PathError`，然后是`e`，我们可以检查它以获取有关错误的更多信息。

## Panic 恐慌

报告错误给调用者的通常方法是返回一个`error`作为额外的返回值。规范的`Read`方法是一个众所周知的实例，它返回一个字节数量和一个`error`。但是如果错误是不可恢复的呢？有时程序根本无法继续。

为此，有一个内置函数`panic`，它实际上会创建一个运行时错误，该错误将停止程序。该函数接受一个任意类型的参数-通常是一个字符串-在程序终止时被打印出来。这也是一种表明发生了不可能的事情的方式，例如退出无限循环。

```go
// A toy implementation of cube root using Newton's method.
func CubeRoot(x float64) float64 {
    z := x/3   // Arbitrary initial value
    for i := 0; i < 1e6; i++ {
        prevz := z
        z -= (z*z*z-x) / (3*z*z)
        if veryClose(z, prevz) {
            return z
        }
    }
    // A million iterations has not converged; something is wrong.
    panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```

这只是一个例子，但真正的库参数应该避免`panic`。如果问题可以被掩盖或解决，那么让事情继续运行总比取消整个程序更好。一个可能的反例是在初始化期间：如果库缺失无法自行设置，可以这么说，`panic`可能是合理的。

```go
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```

## Recover

当调用`panic`时，包括隐式的运行时错误，例如索引切片越界或者类型断言失败，它立即停止当前函数的执行并开始展开`gorountine`的堆栈，在此过程中运行任何延迟的函数。如果展开到达`gorountine`堆栈的顶部，程序就会终止。但是，可以使用内置函数`recover`重新获得对`goroutine`的控制并恢复正常执行。

对`recover`的调用会停止展开并返回传送给`panic`的参数。因为展开时唯一运行的代码是在延迟函数内部，所以`recover`只在延迟函数内部有用。

恢复的一个应用是关闭服务器内失败的`goroutine`，而不杀死其他正在执行的`goroutine`。

```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```

在这个例子中，如果`do(work)`发生恐慌，结果将被记录下来，`goroutine`将干净地退出而不会打扰其他人。在延迟关闭中不需要做任何其他事情，调用`recover`完全处理这个情况。

因为除非直接从延迟函数调用，`recover`只是返回`nil`，所以延迟代码可以调用库例程，这些例程本身使用`panic`和`recover`而不失败。例如，`safelyDo`中的延迟函数可能会在调用`recover`之前调用日志记录函数，并且该日志记录代码将不受恐慌状态的影响运行。

有了我们的恢复模式，`do`函数（以及它调用的任何东西）可以通过调用`panic`干净利落地摆脱任何糟糕的情况。我们可以使用这个想法去简化复杂软件中的错误处理。让我们看一下`regexp`包的理想化版本，它通过调用带有本地错误类型的`panic`来报告解析错误。下面是`Error`的定义、错误方法和`Compile`函数。

```go
// Error is the type of a parse error; it satisfies the error interface.
type Error string
func (e Error) Error() string {
    return string(e)
}

// error is a method of *Regexp that reports parsing errors by
// panicking with an Error.
func (regexp *Regexp) error(err string) {
    panic(Error(err))
}

// Compile returns a parsed representation of the regular expression.
func Compile(str string) (regexp *Regexp, err error) {
    regexp = new(Regexp)
    // doParse will panic if there is a parse error.
    defer func() {
        if e := recover(); e != nil {
            regexp = nil    // Clear return value.
            err = e.(Error) // Will re-panic if not a parse error.
        }
    }()
    return regexp.doParse(str), nil
}
```

如果`doParse`崩溃，则恢复块会将返回值设置为`nil`-延迟函数可以修改命名的返回值。然后，它会对`err`的赋值中通过断言它具有本地类型`Error`来检查问题是否是解析错误。如果没有，类型断言将会失败，导致运行时错误继续堆栈展开，就好像没有中断它一样。这个检查意味着如果发生意外情况，例如索引越界，即使我们使用`panic`和`recover`处理解析错误，代码也会失败。

错误处理到位后，`error`方法（因为它是一个绑定到类型的方法，它很好，甚至很自然，因为它与内置错误类型具有相同的名称）可以轻松报告解析错误，而无需担心手动展开解析堆栈：

```go
if pos == 0 {
    re.error("'*' illegal at start of expression")
}
```

虽然这种模式很有用，但它应该只在包中使用。`Parse`将其内部恐慌调用转移为错误值，它不会向客户端公开恐慌，这是一个很好的规则。

顺便一提，如果发生实际错误，这个`re-panic`习惯用法会更改`panic`值。但是，原始故障和新故障都会出现在崩溃报告中，因此问题的根本原因依然可见。因此，这种简单的`re-panic`方法通常就足够了-毕竟它只是一个崩溃-但是如果你只想显示原始值，你可以编写更多的代码来过滤意外的问题，并用原始错误重新恐慌。

## A web server 一个网络服务器

让我们完成一个完整的go程序，一个web服务器。这实际上是一个网络重新服务器。Google在`chart.apis.google.com`上提供了一项服务，该服务可将数据自动化为图表和图形。但是，它很难以交互方式使用，因为你需要将数据作为查询放入URL。此处的程序为一种数据形式提供了更好的界面：给定一小段文本，它调用图表服务器生成二维码，即对文本进行编码的框矩阵。该图像可以用手机的摄像头抓取并解释。例如，一个URL，无需您将URL输入到手机的小键盘中

这里是完整的程序，解析如下：

```go
package main

import (
    "flag"
    "html/template"
    "log"
    "net/http"
)

var addr = flag.String("addr", ":1718", "http service address") // Q=17, R=18

var templ = template.Must(template.New("qr").Parse(templateStr))

func main() {
    flag.Parse()
    http.Handle("/", http.HandlerFunc(QR))
    err := http.ListenAndServe(*addr, nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }
}

func QR(w http.ResponseWriter, req *http.Request) {
    templ.Execute(w, req.FormValue("s"))
}

const templateStr = `
<html>
<head>
<title>QR Link Generator</title>
</head>
<body>
{{if .}}
<img src="http://chart.apis.google.com/chart?chs=300x300&cht=qr&choe=UTF-8&chl={{.}}" />
<br>
{{.}}
<br>
<br>
{{end}}
<form action="/" name=f method="GET">
    <input maxLength=1024 size=70 name=s value="" title="Text to QR Encode">
    <input type=submit value="Show QR" name=qr>
</form>
</body>
</html>
`
```

到主要部分应该很容易理解。`flag`为我们的服务器设置默认的HTTP端口，模板变量`templ`是最有趣的地方，它构建了一个由服务器执行并显示页面的HTML模板，稍后将会详细介绍。

`main`函数解析标志，并使用我们上面讨论的机制，将函数`QR`绑定到服务器的根路径。然后调用`http.ListenAndServe`启动服务器，它在服务器运行时阻塞。

`QR`只接收包含表单数据的请求，并对名为`s`的表单值中的数据执行模板。

模板包`html/template`功能强大，该程序仅涉及其功能。本质上，它通过替换从传递给`templ.Execute`的数据项派生的元素（在本例中为表单值），即时重写一段HTML文本。在模板文本`templateStr`中，双花括号分隔的部分表示模板操作。从`{{if.}}`到`{{end}}`的部分仅当在当前数据项的值称为`.`（非空）时才执行。即当字符串为空时，这块模板被抑制。

两个片段`{{.}}`表示在网页上显示呈现给模板的数据-查询字符串。HTML模板包会自动提供适当的转义，以便可以安全地显示文本。

模板字符串的其余部分只是页面加载时显示的HTML。

这就是它：几行代码加上一些数据驱动的HTML文本的有用Web服务器。Go足够强大，可以在几行代码中完成很多事情。
