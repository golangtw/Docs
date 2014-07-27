#Effective Go

- [Introduction](#introduction)
    - [Examples](#examples)
- [格式](#formatting)
- [Commentary](#commentary)
- [Names](#names)
    - [Package names](#package-names)
    - [Getters](#getters)
    - [Interface names](#interface-names)
    - [MixedCaps](#mixedCaps)
- [Semicolons](#semicolons)
- [Control structures](#control-structures)
    - [If](#if)
    - [Redeclaration and reassignment](#redeclaration-and-reassignment)
    - [For](#for)
    - [Switch](#switch)
- [Type switch](#type-switch)
- [Functions](#functions)
    - [Multiple return values](#multiple-return-values)
    - [Named result parameters](#named-result-parameters)
    - [Defer](#defer)
- [資料](#data)
    - [new配置](#allocation-with-new)
    - [建構子及字面值組合](#constructors-and-composite-literals)
    - [make配置](#allocation-with-make)
    - [陣列](#arrays)
    - [Slice](#slices)
    - [二維slice](#two-dimensional-slices)
    - [Map](#maps)
    - [印出](#printing)
    - [附加](#append)
- [初始化](#initialization)
    - [常數](#constants)
    - [變數](#variables)
    - [`init`函式](#the-init-function)
- [方法](#methods)
    - [指標與值的比較](#pointers-vs-values)
- [介面及其他型別](#interfaces-and-other-types)
    - [介面](#interfaces)
    - [轉換](#conversions)
    - [介面轉型及型別斷言](#interface-conversions-and-type-assertions)
    - [通用性](generality)
    - [介面及方法](#interfaces-and-methods)
- [The blank identifier](#the-blank-identifier)
    - [The blank identifier in multiple assignment](#the-blank-identifier-in-multiple-assignment)
    - [Unused imports and variables](#unused-imports-and-variables)
    - [Import for side effect](#import-for-side-effect)
    - [Interface checks](#interface-checks)
- [Embedding](#embedding)
- [Concurrency](#concurrency)
    - [以通訊實現資料共享](#share-by-communicating)
    - [Goroutines](#goroutines)
    - [Channels](#channels)
    - [用 channel 傳遞 channel](#channels-of-channels)
    - [平行處理 (Parallelization)](#parallelization)
    - [不會滿溢的緩衝區](#a-leaky-buffer)
- [錯誤處理](#errors)
    - [Panic](#panic)
    - [Recover](#recove)
- [A web server](#a-web-server)

---

##簡介

Go 是一個新的語言。雖然這個語言從其他語言借用了許多想法，但他有許多特點使得高效的 Go 程式與其他語言實作的程式許多不同。 (暫譯)

Go is a new language. Although it borrows ideas from existing languages, it has unusual properties that make effective Go programs different in character from programs written in its relatives. A straightforward translation of a C++ or Java program into Go is unlikely to produce a satisfactory result—Java programs are written in Java, not Go. On the other hand, thinking about the problem from a Go perspective could produce a successful but quite different program. In other words, to write Go well, it's important to understand its properties and idioms. It's also important to know the established conventions for programming in Go, such as naming, formatting, program construction, and so on, so that programs you write will be easy for other Go programmers to understand.

This document gives tips for writing clear, idiomatic Go code. It augments the [language specification](http://golang.org/ref/spec), the [Tour of Go](http://tour.golang.org/), and [How to Write Go Code](http://golang.org/doc/code.html), all of which you should read first.

###Examples

The [Go package sources](http://golang.org/src/pkg/) are intended to serve not only as the core library but also as examples of how to use the language. Moreover, many of the packages contain working, self-contained executable examples you can run directly from the [golang.org](http://golang.org/) web site, such as [this onea](http://golang.org/pkg/strings/#example_Map) (if necessary, click on the word "Example" to open it up). If you have a question about how to approach a problem or how something might be implemented, the documentation, code and examples in the library can provide answers, ideas and background.

##<a name="formatting"></a>格式

格式的問題最容易引起爭論同時也是最不重要的。我們可以適應各種不同的格式，但最好的情況就是不用適應，如果每個人都遵循相同的格式，我們就可以花更少的時間在格式上。所以真正的問題在於要如何不透過冗長的規範手冊達到這樣的烏托邦境界。

在 Go 中我們採用了一種不尋常的方法：讓機器處理大部分的格式問題。gofmt 指令（也可以使用 go fmt ，它是在 package 層級運作而非 source 層級）可以讀取一份 go 程式並產出以標準格式縮進以及垂直排版的原始碼，同時保留注釋的格式，但有必要的話會重排注釋。如果你想要知道如何處理一些從未見過的編排情況，執行 gofmt；如果結果看起來不太對，那就重新整理你的程式（或是提交關於 gofmt 的 bug ），不要使用能避開問題的替代方法。

如同範例所示，不需要花時間對齊 struct 欄位後面的注釋。Gofmt 會幫你處理。宣告一個 struct
```go
    type T struct {
        name string // 物件的名字
        value int // 它的值
    }
```
gofmt 將會對齊欄位
```go
    type T struct {
        name    string // 物件的名字
        value   int    // 它的值
    }
```
所有 Go 標準函式庫中的程式碼都使用了 gofmt 格式化。

這裡還有一些細節需要注意。簡單來講：

縮進
    我們使用 tabs 做為縮進而且讓 gofmt 預設產出它。當真正需要時才使用空白。

行長
    Go 沒有行的長度限制。不用擔心超過穿孔卡片。如果覺得某行太長，換行並且使用額外的縮進。

圓括號
    比起C與Java，Go需要的圓括號更少：控制結構（if, for, switch）在它們的語法中不需要圓括號。同樣的，運算子優先層級可以變得更短更清楚，所以
```go
    x<<8 + y<<16
```
可以明顯表示出間隔所暗示的意義，而不像在其他語言裡一樣隱晦。

##Commentary

Go provides C-style /* */ block comments and C++-style // line comments. Line comments are the norm; block comments appear mostly as package comments, but are useful within an expression or to disable large swaths of code.

The program—and web server—godoc processes Go source files to extract documentation about the contents of the package. Comments that appear before top-level declarations, with no intervening newlines, are extracted along with the declaration to serve as explanatory text for the item. The nature and style of these comments determines the quality of the documentation godoc produces.

Every package should have a package comment, a block comment preceding the package clause. For multi-file packages, the package comment only needs to be present in one file, and any one will do. The package comment should introduce the package and provide information relevant to the package as a whole. It will appear first on the godoc page and should set up the detailed documentation that follows.

    /*
    Package regexp implements a simple library for regular expressions.

    The syntax of the regular expressions accepted is:

        regexp:
            concatenation { '|' concatenation }
        concatenation:
            { closure }
        closure:
            term [ '*' | '+' | '?' ]
        term:
            '^'
            '$'
            '.'
            character
            '[' [ '^' ] character-ranges ']'
            '(' regexp ')'
    */
    package regexp

If the package is simple, the package comment can be brief.

    // Package path implements utility routines for
    // manipulating slash-separated filename paths.

Comments do not need extra formatting such as banners of stars. The generated output may not even be presented in a fixed-width font, so don't depend on spacing for alignment—godoc, like gofmt, takes care of that. The comments are uninterpreted plain text, so HTML and other annotations such as _this_ will reproduce verbatim and should not be used. One adjustment godoc does do is to display indented text in a fixed-width font, suitable for program snippets. The package comment for the fmt package uses this to good effect.

Depending on the context, godoc might not even reformat comments, so make sure they look good straight up: use correct spelling, punctuation, and sentence structure, fold long lines, and so on.

Inside a package, any comment immediately preceding a top-level declaration serves as a doc comment for that declaration. Every exported (capitalized) name in a program should have a doc comment.

Doc comments work best as complete sentences, which allow a wide variety of automated presentations. The first sentence should be a one-sentence summary that starts with the name being declared.

    // Compile parses a regular expression and returns, if successful, a Regexp
    // object that can be used to match against text.
    func Compile(str string) (regexp *Regexp, err error) {

If the name always begins the comment, the output of godoc can usefully be run through grep. Imagine you couldn't remember the name "Compile" but were looking for the parsing function for regular expressions, so you ran the command,

    $ godoc regexp | grep parse

If all the doc comments in the package began, "This function...", grep wouldn't help you remember the name. But because the package starts each doc comment with the name, you'd see something like this, which recalls the word you're looking for.

    $ godoc regexp | grep parse
        Compile parses a regular expression and returns, if successful, a Regexp
        parsed. It simplifies safe initialization of global variables holding
        cannot be parsed. It simplifies safe initialization of global variables
    $

Go's declaration syntax allows grouping of declarations. A single doc comment can introduce a group of related constants or variables. Since the whole declaration is presented, such a comment can often be perfunctory.

    // Error codes returned by failures to parse an expression.
    var (
        ErrInternal      = errors.New("regexp: internal error")
        ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
        ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
        ...
    )

Even for private names, grouping can also indicate relationships between items, such as the fact that a set of variables is protected by a mutex.

    var (
        countLock   sync.Mutex
        inputCount  uint32
        outputCount uint32
        errorCount  uint32
    )

##Names

Names are as important in Go as in any other language. They even have semantic effect: the visibility of a name outside a package is determined by whether its first character is upper case. It's therefore worth spending a little time talking about naming conventions in Go programs.

###Package names

When a package is imported, the package name becomes an accessor for the contents. After

    import "bytes"

the importing package can talk about bytes.Buffer. It's helpful if everyone using the package can use the same name to refer to its contents, which implies that the package name should be good: short, concise, evocative. By convention, packages are given lower case, single-word names; there should be no need for underscores or mixedCaps. Err on the side of brevity, since everyone using your package will be typing that name. And don't worry about collisions a priori. The package name is only the default name for imports; it need not be unique across all source code, and in the rare case of a collision the importing package can choose a different name to use locally. In any case, confusion is rare because the file name in the import determines just which package is being used.

Another convention is that the package name is the base name of its source directory; the package in src/pkg/encoding/base64 is imported as "encoding/base64" but has name base64, not encoding_base64 and not encodingBase64.

The importer of a package will use the name to refer to its contents. so exported names in the package can use that fact to avoid stutter. (Don't use the import . notation, which can simplify tests that must run outside the package they are testing, but should otherwise be avoided.) For instance, the buffered reader type in the bufio package is called Reader, not BufReader, because users see it as bufio.Reader, which is a clear, concise name. Moreover, because imported entities are always addressed with their package name, bufio.Reader does not conflict with io.Reader. Similarly, the function to make new instances of ring.Ring—which is the definition of a constructor in Go—would normally be called NewRing, but since Ring is the only type exported by the package, and since the package is called ring, it's called just New, which clients of the package see as ring.New. Use the package structure to help you choose good names.

Another short example is once.Do; once.Do(setup) reads well and would not be improved by writing once.DoOrWaitUntilDone(setup). Long names don't automatically make things more readable. A helpful doc comment can often be more valuable than an extra long name.

###Getters

Go doesn't provide automatic support for getters and setters. There's nothing wrong with providing getters and setters yourself, and it's often appropriate to do so, but it's neither idiomatic nor necessary to put Get into the getter's name. If you have a field called owner (lower case, unexported), the getter method should be called Owner (upper case, exported), not GetOwner. The use of upper-case names for export provides the hook to discriminate the field from the method. A setter function, if needed, will likely be called SetOwner. Both names read well in practice:

    owner := obj.Owner()
    if owner != user {
        obj.SetOwner(user)
    }

###Interface names

By convention, one-method interfaces are named by the method name plus an -er suffix or similar modification to construct an agent noun: Reader, Writer, Formatter, CloseNotifier etc.

There are a number of such names and it's productive to honor them and the function names they capture. Read, Write, Close, Flush, String and so on have canonical signatures and meanings. To avoid confusion, don't give your method one of those names unless it has the same signature and meaning. Conversely, if your type implements a method with the same meaning as a method on a well-known type, give it the same name and signature; call your string-converter method String not ToString.

###MixedCaps

Finally, the convention in Go is to use MixedCaps or mixedCaps rather than underscores to write multiword names.

##Semicolons

Like C, Go's formal grammar uses semicolons to terminate statements, but unlike in C, those semicolons do not appear in the source. Instead the lexer uses a simple rule to insert semicolons automatically as it scans, so the input text is mostly free of them.

The rule is this. If the last token before a newline is an identifier (which includes words like int and float64), a basic literal such as a number or string constant, or one of the tokens

    break continue fallthrough return ++ -- ) }

the lexer always inserts a semicolon after the token. This could be summarized as, “if the newline comes after a token that could end a statement, insert a semicolon”.

A semicolon can also be omitted immediately before a closing brace, so a statement such as

    go func() { for { dst <- <-src } }()

needs no semicolons. Idiomatic Go programs have semicolons only in places such as for loop clauses, to separate the initializer, condition, and continuation elements. They are also necessary to separate multiple statements on a line, should you write code that way.

One consequence of the semicolon insertion rules is that you cannot put the opening brace of a control structure (if, for, switch, or select) on the next line. If you do, a semicolon will be inserted before the brace, which could cause unwanted effects. Write them like this

    if i < f() {
        g()
    }

not like this

    if i < f()  // wrong!
    {           // wrong!
        g()
    }

##Control structures

The control structures of Go are related to those of C but differ in important ways. There is no do or while loop, only a slightly generalized for; switch is more flexible; if and switch accept an optional initialization statement like that of for; break and continue statements take an optional label to identify what to break or continue; and there are new control structures including a type switch and a multiway communications multiplexer, select. The syntax is also slightly different: there are no parentheses and the bodies must always be brace-delimited.

###If

In Go a simple if looks like this:

    if x > 0 {
        return y
    }

Mandatory braces encourage writing simple if statements on multiple lines. It's good style to do so anyway, especially when the body contains a control statement such as a return or break.

Since if and switch accept an initialization statement, it's common to see one used to set up a local variable.

    if err := file.Chmod(0664); err != nil {
        log.Print(err)
        return err
    }

In the Go libraries, you'll find that when an if statement doesn't flow into the next statement—that is, the body ends in break, continue, goto, or return—the unnecessary else is omitted.

    f, err := os.Open(name)
    if err != nil {
        return err
    }
    codeUsing(f)

This is an example of a common situation where code must guard against a sequence of error conditions. The code reads well if the successful flow of control runs down the page, eliminating error cases as they arise. Since error cases tend to end in return statements, the resulting code needs no else statements.

    f, err := os.Open(name)
    if err != nil {
        return err
    }
    d, err := f.Stat()
    if err != nil {
        f.Close()
        return err
    }
    codeUsing(f, d)

###Redeclaration and reassignment

An aside: The last example in the previous section demonstrates a detail of how the := short declaration form works. The declaration that calls os.Open reads,

    f, err := os.Open(name)

This statement declares two variables, f and err. A few lines later, the call to f.Stat reads,

    d, err := f.Stat()

which looks as if it declares d and err. Notice, though, that err appears in both statements. This duplication is legal: err is declared by the first statement, but only re-assigned in the second. This means that the call to f.Stat uses the existing err variable declared above, and just gives it a new value.

In a := declaration a variable v may appear even if it has already been declared, provided:

- this declaration is in the same scope as the existing declaration of v (if v is already declared in an outer scope, the declaration will create a new variable §),
- the corresponding value in the initialization is assignable to v, and
- there is at least one other variable in the declaration that is being declared anew.

This unusual property is pure pragmatism, making it easy to use a single err value, for example, in a long if-else chain. You'll see it used often.

§ It's worth noting here that in Go the scope of function parameters and return values is the same as the function body, even though they appear lexically outside the braces that enclose the body.

###For

The Go for loop is similar to—but not the same as—C's. It unifies for and while and there is no do-while. There are three forms, only one of which has semicolons.

    // Like a C for
    for init; condition; post { }

    // Like a C while
    for condition { }

    // Like a C for(;;)
    for { }

Short declarations make it easy to declare the index variable right in the loop.

    sum := 0
    for i := 0; i < 10; i++ {
        sum += i
    }

If you're looping over an array, slice, string, or map, or reading from a channel, a range clause can manage the loop.

    for key, value := range oldMap {
        newMap[key] = value
    }

If you only need the first item in the range (the key or index), drop the second:

    for key := range m {
        if key.expired() {
            delete(m, key)
        }
    }

If you only need the second item in the range (the value), use the blank identifier, an underscore, to discard the first:

    sum := 0
    for _, value := range array {
        sum += value
    }

The blank identifier has many uses, as described in a later section.

For strings, the range does more work for you, breaking out individual Unicode code points by parsing the UTF-8. Erroneous encodings consume one byte and produce the replacement rune U+FFFD. (The name (with associated builtin type) rune is Go terminology for a single Unicode code point. See the language specification for details.) The loop

    for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
        fmt.Printf("character %#U starts at byte position %d\n", char, pos)
    }

prints

    character U+65E5 '日' starts at byte position 0
    character U+672C '本' starts at byte position 3
    character U+FFFD '�' starts at byte position 6
    character U+8A9E '語' starts at byte position 7

Finally, Go has no comma operator and ++ and -- are statements not expressions. Thus if you want to run multiple variables in a for you should use parallel assignment (although that precludes ++ and --).

    // Reverse a
    for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
        a[i], a[j] = a[j], a[i]
    }

###Switch

Go's switch is more general than C's. The expressions need not be constants or even integers, the cases are evaluated top to bottom until a match is found, and if the switch has no expression it switches on true. It's therefore possible—and idiomatic—to write an if-else-if-else chain as a switch.

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

There is no automatic fall through, but cases can be presented in comma-separated lists.

    func shouldEscape(c byte) bool {
        switch c {
        case ' ', '?', '&', '=', '#', '+', '%':
            return true
        }
        return false
    }

Although they are not nearly as common in Go as some other C-like languages, break statements can be used to terminate a switch early. Sometimes, though, it's necessary to break out of a surrounding loop, not the switch, and in Go that can be accomplished by putting a label on the loop and "breaking" to that label. This example shows both uses.

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

Of course, the continue statement also accepts an optional label but it applies only to loops.

To close this section, here's a comparison routine for byte slices that uses two switch statements:

    // Compare returns an integer comparing the two byte slices,
    // lexicographically.
    // The result will be 0 if a == b, -1 if a < b, and +1 if a > b
    func Compare(a, b []byte) int {
        for i := 0; i < len(a) && i < len(b); i++ {
            switch {
            case a[i] > b[i]:
                return 1
            case a[i] < b[i]:
                return -1
            }
        }
        switch {
        case len(a) > len(b):
            return 1
        case len(a) < len(b):
            return -1
        }
        return 0
    }

##<a name="type-switch"></a>Type switch

`switch` 區塊也可以用以動態判斷一個 interface 變數的實際型別，寫法上像對該變數作 type assertion，但是把括號內的型別名稱改成關鍵字 `type`。如果 `switch` 開頭的轉型敘述用一個變數來承接，此變數在每個 `case` 述句內就會直接被轉型；更常見的用法，是直接讓承接用的變數與被判斷的原變數同名，如此在各 `case` 述句下，原變數有如被轉型過了一般。

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("非預期的型別 %T", t)              // %T 可以印出 t 的型別名稱
case bool:
    fmt.Printf("boolean %t\n", t)                 // t 的型別是 bool
case int:
    fmt.Printf("整數 %d\n", t)                    // t 的型別是 int
case *bool:
    fmt.Printf("一個指標，指向 boolean %t\n", *t) // t 的型別是 *bool
case *int:
    fmt.Printf("一個指標，指向整數 %d\n", *t)     // t 的型別是 *int
}
```

##Functions
###Multiple return values

One of Go's unusual features is that functions and methods can return multiple values. This form can be used to improve on a couple of clumsy idioms in C programs: in-band error returns such as -1 for EOF and modifying an argument passed by address.

In C, a write error is signaled by a negative count with the error code secreted away in a volatile location. In Go, Write can return a count and an error: “Yes, you wrote some bytes but not all of them because you filled the device”. The signature of the Write method on files from package os is:

    func (file *File) Write(b []byte) (n int, err error)

and as the documentation says, it returns the number of bytes written and a non-nil error when n != len(b). This is a common style; see the section on error handling for more examples.

A similar approach obviates the need to pass a pointer to a return value to simulate a reference parameter. Here's a simple-minded function to grab a number from a position in a byte slice, returning the number and the next position.

    func nextInt(b []byte, i int) (int, int) {
        for ; i < len(b) && !isDigit(b[i]); i++ {
        }
        x := 0
        for ; i < len(b) && isDigit(b[i]); i++ {
            x = x*10 + int(b[i]) - '0'
        }
        return x, i
    }

You could use it to scan the numbers in an input slice b like this:

    for i := 0; i < len(b); {
        x, i = nextInt(b, i)
        fmt.Println(x)
    }

###Named result parameters

The return or result "parameters" of a Go function can be given names and used as regular variables, just like the incoming parameters. When named, they are initialized to the zero values for their types when the function begins; if the function executes a return statement with no arguments, the current values of the result parameters are used as the returned values.

The names are not mandatory but they can make code shorter and clearer: they're documentation. If we name the results of nextInt it becomes obvious which returned int is which.

    func nextInt(b []byte, pos int) (value, nextPos int) {

Because named results are initialized and tied to an unadorned return, they can simplify as well as clarify. Here's a version of io.ReadFull that uses them well:

    func ReadFull(r Reader, buf []byte) (n int, err error) {
        for len(buf) > 0 && err == nil {
            var nr int
            nr, err = r.Read(buf)
            n += nr
            buf = buf[nr:]
        }
        return
    }

###Defer

Go's defer statement schedules a function call (the deferred function) to be run immediately before the function executing the defer returns. It's an unusual but effective way to deal with situations such as resources that must be released regardless of which path a function takes to return. The canonical examples are unlocking a mutex or closing a file.

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

Deferring a call to a function such as Close has two advantages. First, it guarantees that you will never forget to close the file, a mistake that's easy to make if you later edit the function to add a new return path. Second, it means that the close sits near the open, which is much clearer than placing it at the end of the function.

The arguments to the deferred function (which include the receiver if the function is a method) are evaluated when the defer executes, not when the call executes. Besides avoiding worries about variables changing values as the function executes, this means that a single deferred call site can defer multiple function executions. Here's a silly example.

    for i := 0; i < 5; i++ {
        defer fmt.Printf("%d ", i)
    }

Deferred functions are executed in LIFO order, so this code will cause 4 3 2 1 0 to be printed when the function returns. A more plausible example is a simple way to trace function execution through the program. We could write a couple of simple tracing routines like this:

    func trace(s string)   { fmt.Println("entering:", s) }
    func untrace(s string) { fmt.Println("leaving:", s) }

    // Use them like this:
    func a() {
        trace("a")
        defer untrace("a")
        // do something....
    }

We can do better by exploiting the fact that arguments to deferred functions are evaluated when the defer executes. The tracing routine can set up the argument to the untracing routine. This example:

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

prints

    entering: b
    in b
    entering: a
    in a
    leaving: a
    leaving: b

For programmers accustomed to block-level resource management from other languages, defer may seem peculiar, but its most interesting and powerful applications come precisely from the fact that it's not block-based but function-based. In the section on panic and recover we'll see another example of its possibilities.

##<a name="data"></a>資料
###<a name="allocation-with-new"></a>`new`配置

Go有兩種原生配置方法，分別為內建函示`new`及`make`。這兩個方法有差異且適用於不同的型別，這容易讓人混淆，但其實規則很簡單。先說有關`new`。它是一種內建函示，用來配置記憶體。它不像其他語言中的`new`會*初始化*記憶體，它只會給予*空*值。意思是說，`new(T)`會爲`T`型別的新項目分配空的儲存空間，並且傳回其位址，配置出來的是一種`*T`值。在Go術語裡，它傳回的是一個指標，指向一個新分配的`T`型別空值。

因為`new`傳回的記憶體是個空值，所以當你在安排你所設計的資料結構時很有幫助，因為每個型別都是空值，不需要進一步的初始化。意味著，資料結構的使用者可以用`new`來新建並馬上使用。舉例來說，`bytes.Buffer`的文件陳述了"`Buffer`的空值是一種空的且可立即使用的緩衝區。" 同樣地，`sync.Mutex`沒有一種明確的建構子或`Init`方法。換句話說，`sync.Mutex`的空值定義成一種未上鎖的mutex。

"空值可用"的性質是有用的。考慮下列型別宣告。

    type SyncedBuffer struct {
        lock    sync.Mutex
        buffer  bytes.Buffer
    }

`SyncBuffer`型別的值分配之後可以立即使用，或者只是單純宣告它。下一個片段碼中，`p`與`v`兩者都可以正常運作，無須進一步的安排。

    p := new(SyncedBuffer)  // *SyncedBuffer指標型別
    var v SyncedBuffer      //  SyncedBuffer型別

###<a name="constructors-and-composite-literals"></a>建構子及字面合成

有時候只是空值還不夠，初始建構子是必要的，如下範例，源自於`os`包。

    func NewFile(fd int, name string) *File {
        if fd < 0 {
            return nil
        }
        f := new(File)
        f.fd = fd
        f.name = name
        f.dirinfo = nil
        f.nepipe = 0
        return f
    }

這裡有很多樣板。我們可以用字面合成來簡化它，字面合成是一種表達式，每次賦值時都會建立一個新的實例。

    func NewFile(fd int, name string) *File {
        if fd < 0 {
            return nil
        }
        f := File{fd, name, nil, 0}
        return &f
    }

注意，這並不像C，這裡回傳一個區域變數的位址是完全OK的；當函示傳回之後，變數的儲存空間仍然存活著。實際上，每當賦值字面合成取得位址時，都會分配一個新的實例，所以我們可以合併最後兩行。

    return &File{fd, name, nil, 0}

字面合成的字段必須依照順序排列，並且缺一不可。但如果是明確地標上`field:value`這種組合的話，初始時就可以無視順序，缺少的部份就會用它們本身的空值來初始化。因此，我們可以說

    return &File{fd: fd, name: name}

如上述有限的例子中，字面合成並沒有包含所有的字段，它就會為該型別建立空值。`new(File)`表達式與`&File{}`是相同的。

字面合成也可以用來產生陣列、slice及map，而字段就用來表示索引或者map的鍵。下列範例中，初始化了`Enone`、`Eio`及`Einval`值，它們之間是有區別的。

    a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
    s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
    m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}

###<a name="allocation-with-make"></a>`make`配置

回到配置記憶體。內建函式`make(T, args)`提供一種與`new(T)`不同的意圖。它只能用來產生slice、map及channel，而且它傳回`T`型別(非`*T`型別)的值(非空值)。這個區別的意義在於，這三個型別呈現的、底層的資料結構必須在使用之前初始化完畢。舉例來說，slice是擁有三個項目描述子，包括指向資料(陣列)的指標，長度及容量，直到這些項目初始化完畢前，slice將會是個`nil`。對於slice、map及channel而言，`make`初始了內部資料結構，並準備可用的值。例如，

    make([]int, 10, 100)

配置了一個有100個int的陣列，接著產生一個slice結構，長度為10及容量為100，指向陣列的前10個元素。(當產生一個slice時，容量是可以省略的；詳情請見slice章節。) 相較之下，`new([]int)`傳回一個指標，指向一個新配置、空值的slice結構，也就是說，一個指向`nil`的slice指標。

接下來的範例舉出了`new`與`make`之間的差異。

    var p *[]int = new([]int)       // 配置slice結構；*p == nil; 很少有用
    var v  []int = make([]int, 100) // slice v 現在參考到一個有100個int的陣列

    // 複雜多餘：
    var p *[]int = new([]int)
    *p = make([]int, 100, 100)

    // 慣用語法:
    v := make([]int, 100)

記住`make`只適用在map、slice及channel，且不會傳回指標。`new`配置明確地取得指標，若要取得對`make`的值的指標，必須對值取址來取得指標。

###<a name="arrays"></a>陣列

當你計畫著記憶體的詳細布局時，陣列非常有用，有時候可以避免記憶體配置，但主要是用來做slice的建造區塊，這是下一章節的主題。為了這個章節，需要先鋪設一些基礎，這裡有些關於陣列的兩三語。

下列是Go與C之間陣列運作的差異。在Go中，

- 陣列就是個值。指定陣列到另一個陣列，將會複製所有的元素內容。
- 尤其是，如果你在函數參數傳遞到陣列，函數內會使用陣列的副本，而不是指向陣列的指標
- 尺寸也是一種陣列型別。`[10]int`型別與`[20]int`型別是不同的。

值的性質可以非常有用，但也很昂貴；如果你想要像C同樣的行為及效率，你可以傳遞陣列的指標。

    func Sum(a *[3]float64) (sum float64) {
        for _, v := range *a {
            sum += v
        }
        return
    }

    array := [...]float64{7.0, 8.5, 9.1}
    x := Sum(&array)  // 注意，這裡明確地使用取址運算子

但是，這種方式不是Go的慣用風格。請改用slice。

###<a name="slices"></a>Slice

Slice包裝陣列，給予了連續資料更廣泛、強大且方便的界面，除了項目有明確的維度，如矩陣轉置之外，Go裡面大多的陣列程式設計都是用slice來完成，而非一般的陣列。

Slice持有底層陣列的參考，如果你將slice指定到另一個slice，兩著會參考到相同的陣列。如果某個函式接受slice參數，元素的修改也會反應到呼叫者本身的slice，類似於傳遞指向底層陣列的指標。`Read`函式接受slice參數而不是指標及數量；slice內的長度設定了多少資料可以讀取的上限。下列是`File`型別的`Read`方法的函數簽名，源自`os`包：

    func (file *File) Read(buf []byte) (n int, err error)

此方法回傳讀取了多少bytes，如果錯誤發生的話，傳回錯誤值。下列示範從一個大的緩衝`b`讀取前32個bytes，進行緩衝切片。

        n, err := f.Read(buf[0:32])

這類的切片常見且有效率。實際上，先不管效率，下列片段碼也會讀取緩衝的前32個bytes。

    var n int
    var err error
    for i := 0; i < 32; i++ {
        nbytes, e := f.Read(buf[i:i+1])  // 讀取1個byte.
        if nbytes == 0 || e != nil {
            err = e
            break
        }
        n += nbytes
    }

slice的長度可以不斷的改變，只要符合底層陣列的上限；重複指派slice本身。slice的*容量*可以透過內建函式`cap`來存取，回報slice的最大長度。這裡有個函式用來附加資料到slice。如果超過容量，slice將會重新配置。新的slice會被傳回。這個函式可以用在`nil`的slice並且傳回0。

    func Append(slice, data[]byte) []byte {
        l := len(slice)
        if l + len(data) > cap(slice) {  // 重新配置
            // 為了將來成長，配置我們所需量的2倍
            newSlice := make([]byte, (l+len(data))*2)
            // copy函式是預先宣告好的，並且可以用在任何slice型別
            copy(newSlice, slice)
            slice = newSlice
        }
        slice = slice[0:l+len(data)]
        for i, c := range data {
            slice[l+i] = c
        }
        return slice
    }

我們最終必須回傳slice，因為，儘管Append可以修改slice內的元素，但是slice(執行時期的資料結構，持有著指標、長度及容量)是透過傳值方式傳入。  
(譯註：Append函式是以傳值方式傳遞slice，而非傳址，修改的結果需要反應到呼叫者)

附加在slice的這個主意非常有幫助，獲得內建append函式的青睞。為了進一步了解函式的設計、想法，我們需要一些些額外資訊，所以稍候再回來。

###<a name="two-dimensional-slices"></a>二維slice

Go的陣列及slice都是一維的。若要產生二維的陣列或slice，需要定義一個陣列的陣列或slice的slice，像是：

    type Transform [3][3]float64  // 3x3陣列，真正的陣列的陣列。
    type LinesOfText [][]byte     // 一個slice中每個元素都是byte slice。

因為slice是動態長度，所以每個內部的slice都可能是不同長度。這個狀況很常見，如同我們上述的`LinesOfText`：每行都有各自的文字長度。

    text := LinesOfText{
        []byte("Now is the time"),
        []byte("for all good gophers"),
        []byte("to bring some fun to the party."),
    }

有些時候有必須配置二維slice，例如，這種狀況會發生在，當你要掃描處理每行像素的時候。兩個可以達到需求的方式。一是獨立地的為每個slice配置；另一個是配置一個一維陣列，並將各個slice的指標存入該陣列元素中。至於要使用哪種方式，取決於你的應用程式。如果slice會成長或收縮，那就應該用獨立地的配置來避免覆寫到下一行的內容；如果不是，那建立一個物件只有單一配置會來得有效率一點。以下兩個方法的草圖供您參考。首先，一次一行：

    // 配置最上層slice。
    picture := make([][]uint8, YSize) // 每一行都有y個單位。
    // 循環每一行，配置個別slice。
    for i := range picture {
        picture[i] = make([]uint8, XSize)
    }

另一種，一個配置，根據每行進行切片：

    // 配置最上層slice，相同於上個範例。
    picture := make([][]uint8, YSize) // 每一行都有y個單位。
    // 配置一個大的slice，持有所有像素。
    pixels := make([]uint8, XSize*YSize) // 這裡用[]uint8型別，儘管picture的是[][]uint8型別。
    // 循環每一行，從剩下的pixels開頭切出每一行的長度的切片
    for i := range picture {
        picture[i], pixels = pixels[:XSize], pixels[XSize:]
    }

###<a name="maps"></a>Map

Map是種方便且強大的內建資料結構，每個型別的值(鍵)聯繫對應著另一個型別的值(元素或值)組成一對。鍵可以是任何已經定義等號操作子的型別，例如整數、浮點數、複數、字串、指標、界面(只要動態型別支援等號)、結構及陣列。slice型別並不能成為Map的鍵，因為它沒有定義等號。如同slice，map持有底層資料結構的參考。如果你傳遞map到函式之中，函式內對map的內容的修改，也會反應到呼叫者。

Map通常使用字面合成的語法來建構，語法是一種冒號分隔的鍵與值成對組合，所以在初始化時很容易建造它們。

    var timeZone = map[string]int{
        "UTC":  0*60*60,
        "EST": -5*60*60,
        "CST": -6*60*60,
        "MST": -7*60*60,
        "PST": -8*60*60,
    }

指派或取得map值看似語意的，只要用類似陣列或slice方式來存取，只是索引值並不一定是整數。

    offset := timeZone["EST"]

嘗試取得不存在的鍵時，map會傳回該鍵型別的空值。例如，如果map包含了整數，查找一個不存在的鍵，將會傳回`0`。集合可以用map來實作，其值的型別為布林值。設定map條目成`true`來放入某個值到集合內，接著透過簡單的索引來測試它。

    attended := map[string]bool{
        "Ann": true,
        "Joe": true,
        ...
    }

    if attended[person] { // 如果person不存在於map之中，將會傳回false
        fmt.Println(person, "was at the meeting")
    }

有時你需要分辨「條目是否存在」與「條目儲存的值是否為零」之間的不同。以下例來說，map裡`"UTC"`這個條目值本來就是零？或其實根本不存在`"UTC"`這個條目？我們可以用多重指派形式來區別這兩種情況。

    var seconds int
    var ok bool
    seconds, ok = timeZone[tz]

很明顯地，這個用法的習慣用語叫做“comma ok”。在這個範例中，如果`tz`存在，`seconds`將會適當地的設定，而`ok`將會是`true`；如果不存在，`sconeds`將會被設定為空值，而`ok`將會是`false`。下列這個函式將它們放在一起，帶有良好的錯誤報告：

    func offset(tz string) int {
        if seconds, ok := timeZone[tz]; ok {
            return seconds
        }
        log.Println("unknown time zone:", tz)
        return 0
    }

只想測試是否存在於map中，實際的值並不關心時，你可以用[空白識別符號](#blank) (`_`) 來取代本來要用來存放值的變數。

    _, present := timeZone[tz]

若要刪除map條目，使用內建函式`delete`，參數為map實例及要刪除的鍵。這是個安全的作法，甚至鍵已經不存在map內。

    delete(timeZone, "PDT")  // 現在是標準時間

###<a name="printing"></a>印出

Go的格式化輸出是使用一種相似於C的`printf`家族風格，但更加地豐富且廣泛。函式存放在`fmt`包，並且為字母大寫：`fmt.Printf`、`fmt.Fprintf`、`fmt.Sprintf`等。字串函式(`Sprinf`等)傳回一個字串，而不是填放在所提供的緩衝中。

你可以不需要提供格式字串。每個`Printf`、`Fprintf`及`Sprintf`都有另一對函式存在，例如`Print`及`Println`。這些函式不需要格式字串，而是用預設格式來產生。`Print`版本會在每個字串型別參數之間，加上空白，`Println`的版本會插入空行。下列範例每一行都會產生相同的結果。

    fmt.Printf("Hello %d\n", 23)
    fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
    fmt.Println("Hello", 23)
    fmt.Println(fmt.Sprint("Hello ", 23))

格式化印出函式`fmt.Fprint`及相關函式，接受第一個參數可以是任何實作`io.Writer`界面的物件；變數`os.Stdout`及`os.Stderr`都是我們所熟悉的實例。

這裡是與C分叉的開始。首先，數字的格式如`%d`並不接受有號無號數或者尺寸的旗標；取而代之的是，利用參數型別來決定該性質。

    var x uint64 = 1<<64 - 1
    fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))

印出

    18446744073709551615 ffffffffffffffff; -1 -1

如果你只是想要預設轉換，例如十進位顯示整數，你可以用包羅萬象的格式`%v`(v表示value)；結果會跟`Print`與`Println`一模一樣。此外，這個格式可以印出任何值，甚至是陣列、slice、結構及map。下列範例印出前個章節定義的時間區間的map。

    fmt.Printf("%v\n", timeZone)  // 或者用 fmt.Println(timeZone)

得到結果

    map[CST:-21600 PST:-28800 EST:-18000 UTC:0 MST:-25200]

當然，map內的鍵會以任何順序呈現。當印出一個結構時，修改版的格式`%+v`會在結果中評註字段名，其他任何的值，也可以用替代格式`%#v`印出以顯示完整的值，並以Go語句呈現。

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

印出

    &{7 -2.35 abc   def}
    &{a:7 b:-2.35 c:abc     def}
    &main.T{a:7, b:-2.35, c:"abc\tdef"}
    map[string] int{"CST":-21600, "PST":-28800, "EST":-18000, "UTC":0, "MST":-25200}

(注意&符號。) 當應用在字串型別或`[]byte`型別的值時，也可以透過`%q`來印出雙引號字串格式。替代格式`%#q`將會使用倒引號取代。(`%q`格式也適用於整數及`rune`型別，產生出一個單引號包住的`rune`常數。) 並且，`%x`可以用在字串、byte陣列、byte slice及整數，產生出一個十六進位表示的長字串, 若帶有空白在格式中 (`% x`)，它將會在每個bytes之間加上空白。

另一個便利的格式為`%T`，印出值的型別。

    fmt.Printf("%T\n", timeZone)

印出

    map[string] int

如果你想要控制自訂型別的預設格式，只需要定義一個`String() string`簽名的函式於該型別中。舉例一個簡單`T`型別，看起來如下。

    func (t *T) String() string {
        return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
    }
    fmt.Printf("%v\n", t)

印出該格式

    7/-2.35/"abc\tdef"

(如果你需要印出`T`型別的值，以及`T`型別的指標，`String`方法的接收者必須是該型別值；上列範例使用指標是因為這樣比較有效率，也是結構型別的慣用方法。詳細請見下方 pointers vs. value receivers 章節)

我們的`String`方法允許呼叫`Sprintf`，因為印出程序完全再進入，而且可以這樣包裝。另外關於這個方法有個重要的細節需要了解，無論如何：不要在`String`方法內呼叫`Sprintf`，它會不確定地重新進到你的`String`方法。這將會發生在當呼叫`Sprintf`時，直接將接收者當作字串直接印出，這樣一來會再一次調用方法。這是個常見且容易犯的錯誤，如下範例所示。

    type MyString string

    func (m MyString) String() string {
        return fmt.Sprintf("MyString=%s", m) // 錯誤：將會永遠重新復發。
    }

這問題很容易解決：轉換參數成基本的字串型別，基本的字串型別沒有`String`方法。

    type MyString string
    func (m MyString) String() string {
        return fmt.Sprintf("MyString=%s", string(m)) // OK: 注意轉換。
    }

在「初始化」章節我們將會看到其他技巧來避免這樣的遞迴。

另一種印出技巧是傳遞某個印出程序直地給另一個程序。`Printf`的簽名使用`...interface{}`型別作為其最終參數，指定了任意長度的參數(任意型別)，可以在顯示格式之後。

    func Printf(format string, v ...interface{}) (n int, err error) {

在`Printf`函式之中，`v`產生一個參數，其型別為`[]interface{}`，但如果它傳遞到另一個接受任意長度參數的函式，那它就會產生像個一般常規的參數清單。下列範例是我們稍早用到的 `log.Println`函式實作。它將它自己的參數直接傳遞到`fmt.Sprintln`以達到實際的字串格式化。

    // Println 當下使用 fmt.Println 將結果印出到標準紀錄者。
    func Println(v ...interface{}) {
        std.Output(2, fmt.Sprintln(v...))  // 用參數做輸出 (int, string)
    }

我們在呼叫`Sprintln`的參數`v`後面加上`...`，告訴編譯器將`v`視為一種參數清單；否則它將會將`v`當作一個slice參數傳遞。

還有更多我們這裡沒有涵蓋到的印出方式。請見`fmt`包在godoc文件中的詳細說明。

順便提及，`...`參數可以是特定型別，例如，`min`函式的`...int`將會選出整數清單內最小的值：

    func Min(a ...int) int {
        min := int(^uint(0) >> 1)  // 最大整數
        for _, i := range a {
            if i < min {
                min = i
            }
        }
        return min
    }

###<a name="append"></a>附加

現在，我們有個遺失的片段，就是需要解釋內建`append`函式的設計。`append`函式簽名跟我們上述的`Append`函式不同。概要地，它看起來是：

    func append(slice []T, elements ...T) []T

其中`T`是一個佔位符，用來表示任何型別。你實際上無法在Go中寫出一個函式，其`T`型別是由呼叫者來決定。這就是為何它是內建的：它必須由編譯器來支援。

`append`就是將資料附加到slice最後並傳回結果。結果必須回傳，因為如同我們自己手寫的`Append`，底層陣列可能會改變。這是個簡單範例

    x := []int{1,2,3}
    x = append(x, 4, 5, 6)
    fmt.Println(x)

印出`[1 2 3 4 5 6]`。所以`append`運作有點像`Printf`，收集任意長度的參數。

但是，如果我們想要像`Append`做的事情一樣，附加一個slice到slice之中呢？簡單：在呼叫端使用`...`，如同我們在上述的輸出所做一樣。下列片段碼產生跟上一個例子一樣的結果。

    x := []int{1,2,3}
    y := []int{4,5,6}
    x = append(x, y...)
    fmt.Println(x)

如果沒有`...`，將無法通過編譯，因為型別會錯；`y`不是`int`型別。

##<a name="initialization"></a>初始化

初始化雖然在GO與C或C++之間，看起來沒有特別不一樣，但在Go中是很強大的。初始化可以用來建構複雜的結構，無論是初始化物件時的排序問題，或者是跨不同的包，都可以正確的處理。

###<a name="constants"></a>常數

常數在Go中就只是個常數。在編譯期間被建立，甚至是定義成函式內的區域變數，常數只能是數字、字元(rune)、字串或布林值。因為編譯期間的限制，常數的定義必須是常數表達式，經由編譯器估值。舉例來說，`1<<3`是個常數表達式，而`math.Sin(math.Pi/4)`不是，因為函式必須在執行時期才能進行呼叫。

在Go中，列舉型常數是使用`iota`列舉器來建立。`itoa`可以是一部份的表達式，也可以用來隱含表示反複意味，它可以很容易的建構出一組複雜的數值。

    type ByteSize float64

    const (
        _           = iota // 利用空白識別符，可以省略這裡的第一個值
        KB ByteSize = 1 << (10 * iota)
        MB
        GB
        TB
        PB
        EB
        ZB
        YB
    )

附加方法到結構中，例如：附加`String`方法到任何一個使用者所自訂的型別，使它能夠根據本身的值來印出結果。雖然在結構上經常看到這樣的應用，但這個技巧對於數值型別也是很有用，例如像是`ByteSize`這樣的浮點數型別。

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

表達式`YB`印出`1.00YB`，而`ByteSize(1e13)`會印出`9.09TB`。

這裡所使用的`Sprintf`來實作`ByteSize`的`String`方法是安全的(避免遞迴引用)，但不是因為轉換，而是因為它在`Sprintf`內使用`%f`，這個並不是個字串格式：當`Sprintf`要一個字串的時候，才會呼叫`String`方法，而`%f`要的是浮點數值。

###<a name="variables"></a>變數

變數可以像常數那樣的初始化，不同的是，初始化可以是一般的表達式，在執行時期進行計算。

    var (
        home   = os.Getenv("HOME")
        user   = os.Getenv("USER")
        gopath = os.Getenv("GOPATH")
    )

###<a name="the-init-function"></a>`init`函式

最終，每個原始檔都可以定義它自己的`init`函式，用來設定必要的初始狀態。(實際上，原始檔中允許多個`init`函式。) 而最終表示最後：`init`會在包中所有變數宣告都已經初始化之後，才會被呼叫，且匯入的包會被最先初始化之後才開始變數的初始化。

除了初始化不能使用表達式做為宣告之外，一種常見的`init`函式用法是在實際執行開始之前，用來檢驗或修復程式狀態。

    func init() {
        if user == "" {
            log.Fatal("$USER 沒有設定")
        }
        if home == "" {
            home = "/home/" + user
        }
        if gopath == "" {
            gopath = home + "/go"
        }
        // gopath 可透過命令列的 --gopath 參數旗標來覆寫這個設定。
        flag.StringVar(&gopath, "gopath", gopath, "覆寫預設的GOPATH")
    }

##<a name="methods"></a>方法
###指標與值的比較

如同我們所見的`ByteSize`，任何具名型別都(named type)可以定義方法(除了指標或介面)；接收者並不需要是個結構。

在前面討論切片的章節中，我們撰寫了一個`Append`函示。我們可以將它定義成一個切片的方法。為了達成目的，首先宣告一個具名型別，讓我們可以綁定這個方法到型別，接著用這個型別的值，作為方法的接收者。

    type ByteSlice []byte

    func (slice ByteSlice) Append(data []byte) []byte {
        // 如同先前的內容
    }

若要取得更新後的切片，需要傳回更新後的結果。我們可以免除掉這難用的設計，重新去定義方法，改用`ByteSlice`的*指標*作為方法的接收者，這樣就可以在方法內直接覆寫呼叫者的切片。

    func (p *ByteSlice) Append(data []byte) {
        slice := *p
        // 如同先前的內容，但不再需要return.
        *p = slice
    }

事實上，我們可以做的更好。修改函式讓它看起來更像標準的`Write`方法，就像是，

    func (p *ByteSlice) Write(data []byte) (n int, err error) {
        slice := *p
        // 再一次，如同先前的內容
        *p = slice
        return len(data), nil
    }

這樣一來，`*ByteSlice`型別滿足了標準的`io.Writer`介面，這樣很方便。例如，我們可以印出。

    var b ByteSlice
    fmt.Fprintf(&b, "This hour has %d days\n", 7)

因為只有`*ByteSlice`滿足了`io.Writer`介面，所以需要傳入`ByteSlice`的位址。指標與值的規則是，值的方法能夠被指標或值接收者所調用，而指標的方法只有指標接收者可以調用。因為指標的方法可以修改接收者；在值的方法內修改接收者只會改到值的副本，最終會被捨棄。

順道一說，`bytes.Buffer`實作就是對切片位元組使用`Write`的概念。

##<a name="interfaces-and-other-types"></a>介面及其他型別
###<a name="interfaces"></a>介面

Go的介面提供一種方式來指定物件的行為：如果某個東西可以做到*這*，那它就可以用在*這*。我們已經看過幾個範例；可以透過實作`String`方法來自訂印出，`Fprintf`透過`Write`方法來產生輸出。在Go的程式碼中，很常見到介面只有一個或兩個方法，通常會藉此衍生其方法的名稱，例如滿足`io.Writer`介面需要實作`Write`方法。

一個型別可以實作多個介面。例如，某一個集合實作了`sort.Interface`介面，就可以用`sort`包的函式進行排序，介面中包含了`Len()`、`Less(i, j int) bool`及`Swap(i, j int)`這三個方法，也可以自訂格式化。下面人為的範例`Sequence`兩者皆滿足。

    type Sequence []int

    // sort.Interface介面必須的方法。
    func (s Sequence) Len() int {
        return len(s)
    }
    func (s Sequence) Less(i, j int) bool {
        return s[i] < s[j]
    }
    func (s Sequence) Swap(i, j int) {
        s[i], s[j] = s[j], s[i]
    }

    // 印出用的方法 - 印出前排序。
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

###<a name="conversions"></a>轉換

`Sequence`的`String`方法重新建立了`Sprint`已經對切片所提供的工作。如果在呼叫`Sprintf`之前，將`Sequence`轉換成`[] int`，我們可以共享成果。

    func (s Sequence) String() string {
        sort.Sort(s)
        return fmt.Sprint([]int(s))
    }

這也是另一個範例，示範一種轉換技巧，能夠安全的從`String`方法中呼叫`Sprintf`。只要我們忽略掉型別名稱，這兩個型別(`Sequence`及`[] int`)其實是一樣的，兩者之間的轉換是合法的。這樣的轉換不會產生新的值，只是暫時的當作值有新的型別。(也有其他合法的轉換，例如將整數轉換成浮點數，但這會產生新的值。)

Go的程式有一種慣用方法，用來轉換型別表達式，以便存取另一個不同的方法集。如上述範例，我們可以使用既有的`sort.IntSlice`型別，將範例縮減成：

    type Sequence []int

    // 印出用的方法 - 印出前排序元素。
    func (s Sequence) String() string {
        sort.IntSlice(s).Sort()
        return fmt.Sprint([]int(s))
    }

現在，`Sequence`不需要實作多個介面(排序及印出)，我們利用了一種資料項目的能力，轉換多個型別(`Sequence`、`sort.IntSlice`及`[] int`)，各個型別各自完成部分的工作。在實際上比較不常見，但是很有效率。

###<a name="interface-conversions-and-type-assertions"></a>介面轉換及型別斷言

[型別切換](http://golang.org/doc/effective_go.html#type_switch)(type switch)是一種轉換的形式：接受一個介面，切換每一種`case`，某種意義上的型別轉換。下列是一個`fmt.Printf`的簡單化版本，展示如何利用型別切換來將一個值轉換成字串。如果已經是個字串了，我們希望是介面實際持有的字串值，若該型別本身有`String`方法，則希望取得呼叫該方法之後的結果。

    type Stringer interface {
        String() string
    }

    var value interface{} // 由呼叫者提供值。
    switch str := value.(type) {
    case string:
        return str
    case Stringer:
        return str.String()
    }

第一個`case`尋找一個具體的值；第二個`case`將原本的介面轉換到另一個介面。這種混和型別的方式完全沒問題。

但是，如果我們只在意某一種型別呢？如果我們知道這個值持有的是一個`string`，希望取出這個字串呢？可以用只有一個`case`的型別切換，但需要一個*型別斷言*(type assertion)。型別斷言接受一種介面的值，取出值變成一種指定的明確型別。借用來自型別切換的語法，改用型別來取代`type`關鍵字：

    value.(typeName)

其結果是一個新的值，型別是個靜態型別`typeName`。這個型別必須是介面所持有的具體型別，或者是新的值能夠轉換的第二介面型別。我們知道這個值是我們要的字串，我們可以寫成：

    str := value.(string)

但如果這個值沒有包含字串，則會發生執行時期錯誤，導致程式崩潰。為了保護程式，使用慣用的"comma, ok"進行測試，安全地判斷這個值是否是個字串：

    str, ok := value.(string)
    if ok {
        fmt.Printf("字串為: %q\n", str)
    } else {
        fmt.Printf("這個值不是字串\n")
    }

如果型別斷言失敗，`str`將會仍然存在並成為字串型別，但只會是個空值(zero value)，空字串(empty string)。

如上述所描述的能力，下列是一個`if-else`敘述，相等於本章節開頭的型別切換。

    if str, ok := value.(string); ok {
        return str
    } else if str, ok := value.(Stringer); ok {
        return str.String()
    }

###通用性

如果某個型別的存在是為了實作某個介面，而該介面沒有輸出任何方法，那就不需要輸出型別。輸出只是清楚介面的行為，而不是實作，其他不同性質的實作可以映像原始型別的行為。這也能夠避免對每個通用方法的實例重複產出文件。

這種情況下，建構子應該回傳一個介面值，而不是實作的型別。舉例來說，hash函式庫中，`crc32.NewIEEE`及`adler32.New`回傳了`hash.Hash32`介面。Go程式內取代Alder-32的CRC-32演算法只需要更改建構子呼叫；並不會影響剩下的程式碼。

相似的方法允許`crypto`包內各種串流加密演算法，分割串接一起的加密區塊。`crypto/cipher`包內的`Block`介面指明了加密區塊的行為，提供單一區塊的資料加密。接著，同樣在`bufio`包內，加密包實作了`Stream`介面，可以用來建構加密串流，而不需要知道加密區塊的細節。

`crypto/cipher`介面看起來像是：

    type Block interface {
        BlockSize() int
        Encrypt(src, dst []byte)
        Decrypt(src, dst []byte)
    }

    type Stream interface {
        XORKeyStream(dst, src []byte)
    }

下面是計數模式(CTR)串流的定義，將一個區塊加密轉變成一個串流加密；注意區塊加密的細節已經抽象化：

    // NewCTR 回傳一個串流，計數模式下，使用給定的區塊進行加/解密。
    // iv的長度必須與block尺寸相同。
    func NewCTR(block Block, iv []byte) Stream

`NewCTR`不只適用在某一個特定的解密演算法及來源資料，而是任何`Block`介面及`Stream`的實作都適用。因為回傳了介面值，使用其他加密模式來替換CTR只是個局部修改。建構子必須編輯，但因為周遭的程式碼只視結果為`Stream`，所以並不會發現其差異。

###<a name="interfaces-and-methods"></a>介面及方法

幾乎所有東西都可以附加方法，而幾乎所有東西可以滿足一個介面。在`http`包內，`Handler`介面是一個說明性的範例。任何物件只要實作了`Handler`介面，就可以服務HTTP請求。

    type Handler interface {
        ServeHTTP(ResponseWriter, *Request)
    }

`ResponseWriter`是個介面，提供相關所需要的方法以回應客戶端。這些方法中包含標準的`Write`方法，所以`http.ResponseWriter`可以用來當作`io.Writer`使用。`Request`是一種結構，包含了一個已剖析的代表，用來表示來自客戶端的請求。

簡單來說，讓我們先忽略POST方式並假設HTTP的請求一直都是使用GET方式；這樣的簡化不會影響設置控制器(handler)的方式。下列是個瑣碎但是個完整控制器實作，用來計算頁面瀏覽的次數。

    // 簡單計數器伺服器。
    type Counter struct {
        n int
    }

    func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
        ctr.n++
        fmt.Fprintf(w, "次數 = %d\n", ctr.n)
    }

(注意到`Fprintf`可以輸出到`http.ResponseWriter`) 下述範例供你參考，示範如何將上述範例附加到URL樹中成為其中一個分支點。

    import "net/http"
    ...
    ctr := new(Counter)
    http.Handle("/counter", ctr)

但為什麼要讓`Counter`成為一個結構呢？這裡只需要一個數值就可以完成工作。(接收者必須是指標，才能對呼叫者的值進行遞增。)

    // 簡單計數器伺服器。
    type Counter int

    func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
        *ctr++
        fmt.Fprintf(w, "counter = %d\n", *ctr)
    }

如果你希望在程式有一些內部狀態，當一個頁面被瀏覽時能夠收到通知？綁定一個頻道(channel)到網頁。

    // 每次瀏覽時，發送一個通知到頻道。
    // (或許希望頻道有緩衝。)
    type Chan chan *http.Request

    func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
        ch <- req
        fmt.Fprint(w, "notification sent")
    }

最後，假設我們希望能夠在調用二進制的伺服器時，將參數(arguments)呈現於`/args`。寫個函式印出參數很容易。

    func ArgServer() {
        fmt.Println(os.Args)
    }

那我們要怎麼將他變成一個HTTP伺服器呢？我們可以將`ArgServer`變成某個型別的方法，忽略其值，但這裡有更乾淨的作法。除了指標及介面之外，我們可以對任何型別定義方法，所以我們可以為函式寫一個方法。`http`包裡包含以下程式碼：

    // HandlerFunc 型別是一種轉接器，讓我們可以使用一般的函式來作為HTTP的控制器。
    // 如果 f 是一個函式，而且是適當的標記式(singature)，HandlerFunc(f)　就是一個控制器的物件，
    // 其物件會呼叫f函式。
    type HandlerFunc func(ResponseWriter, *Request)

    // ServeHTTP 呼叫 f(c, req).
    func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
        f(w, req)
    }

`HandlerFunc`是一種型別，帶有`ServeHTTP`方法，所以該型別的值可以服務HTTP的請求。看看方法的實作：接收者是個函式，名稱為`f`，方法內呼叫`f`。這可能看起來怪怪的，但它跟方法接收者是個頻道，在方法內傳送頻道，沒有什麼差異。

為了讓`ArgServer`變成HTTP伺服器，首先修改函式，讓它的標記式符合。

    // 參數伺服器。
    func ArgServer(w http.ResponseWriter, req *http.Request) {
        fmt.Fprintln(w, os.Args)
    }

現在`ArgServer`的標記式與`HandlerFunc`相同，所以可以轉換成`HandlerFunc`以存取該方法，就如同我們轉換`Sequence`到`IntSlice`以存取`IntSlice.Sort`方法一樣。如下程式碼，十分簡潔：

    http.Handle("/args", http.HandlerFunc(ArgServer))

當有人瀏覽了 `/args`　頁面，安裝在頁面的控制器有`ArgServer`的值及`HandlerFunc`型別。HTTP伺服器將調用該型別的`ServeHTTP`方法，而`ArgServer`為接收者，該方法接著會呼叫`ArgServer`函式 (位於`HandlerFunc.ServeHTTP`內，透過`f(c, req)`的調用)。參數將會顯示出來。

這個章節中，我們建立了各種HTTP伺服器，包括結構、整數、頻道及函式，因為(幾乎)任何型別都可以定義該介面方法。

##The blank identifier

We've mentioned the blank identifier a couple of times now, in the context of for range loops and maps. The blank identifier can be assigned or declared with any value of any type, with the value discarded harmlessly. It's a bit like writing to the Unix /dev/null file: it represents a write-only value to be used as a place-holder where a variable is needed but the actual value is irrelevant. It has uses beyond those we've seen already.

###The blank identifier in multiple assignment

The use of a blank identifier in a for range loop is a special case of a general situation: multiple assignment.

If an assignment requires multiple values on the left side, but one of the values will not be used by the program, a blank identifier on the left-hand-side of the assignment avoids the need to create a dummy variable and makes it clear that the value is to be discarded. For instance, when calling a function that returns a value and an error, but only the error is important, use the blank identifier to discard the irrelevant value.

    if _, err := os.Stat(path); os.IsNotExist(err) {
        fmt.Printf("%s does not exist\n", path)
    }

Occasionally you'll see code that discards the error value in order to ignore the error; this is terrible practice. Always check error returns; they're provided for a reason.

    // Bad! This code will crash if path does not exist.
    fi, _ := os.Stat(path)
    if fi.IsDir() {
        fmt.Printf("%s is a directory\n", path)
    }

###Unused imports and variables

 It is an error to import a package or to declare a variable without using it. Unused imports bloat the program and slow compilation, while a variable that is initialized but not used is at least a wasted computation and perhaps indicative of a larger bug. When a program is under active development, however, unused imports and variables often arise and it can be annoying to delete them just to have the compilation proceed, only to have them be needed again later. The blank identifier provides a workaround.

This half-written program has two unused imports (fmt and io) and an unused variable (fd), so it will not compile, but it would be nice to see if the code so far is correct.

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

To silence complaints about the unused imports, use a blank identifier to refer to a symbol from the imported package. Similarly, assigning the unused variable fd to the blank identifier will silence the unused variable error. This version of the program does compile.

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

By convention, the global declarations to silence import errors should come right after the imports and be commented, both to make them easy to find and as a reminder to clean things up later.

###Import for side effect

An unused import like fmt or io in the previous example should eventually be used or removed: blank assignments identify code as a work in progress. But sometimes it is useful to import a package only for its side effects, without any explicit use. For example, during its init function, the net/http/pprof package registers HTTP handlers that provide debugging information. It has an exported API, but most clients need only the handler registration and access the data through a web page. To import the package only for its side effects, rename the package to the blank identifier:

    import _ "net/http/pprof"

This form of import makes clear that the package is being imported for its side effects, because there is no other possible use of the package: in this file, it doesn't have a name. (If it did, and we didn't use that name, the compiler would reject the program.)

###Interface checks

As we saw in the discussion of interfaces above, a type need not declare explicitly that it implements an interface. Instead, a type implements the interface just by implementing the interface's methods. In practice, most interface conversions are static and therefore checked at compile time. For example, passing an *os.File to a function expecting an io.Reader will not compile unless *os.File implements the io.Reader interface.

Some interface checks do happen at run-time, though. One instance is in the encoding/json package, which defines a Marshaler interface. When the JSON encoder receives a value that implements that interface, the encoder invokes the value's marshaling method to convert it to JSON instead of doing the standard conversion. The encoder checks this property at run time with a type assertion like:

    m, ok := val.(json.Marshaler)

If it's necessary only to ask whether a type implements an interface, without actually using the interface itself, perhaps as part of an error check, use the blank identifier to ignore the type-asserted value:

    if _, ok := val.(json.Marshaler); ok {
        fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
    }

One place this situation arises is when it is necessary to guarantee within the package implementing the type that it actually satisfies the interface. If a type—for example, json.RawMessage—needs a custom JSON representation, it should implement json.Marshaler, but there are no static conversions that would cause the compiler to verify this automatically. If the type inadvertently fails to satisfy the interface, the JSON encoder will still work, but will not use the custom implementation. To guarantee that the implementation is correct, a global declaration using the blank identifier can be used in the package:

    var _ json.Marshaler = (*RawMessage)(nil)

In this declaration, the assignment involving a conversion of a *RawMessage to a Marshaler requires that *RawMessage implements Marshaler, and that property will be checked at compile time. Should the json.Marshaler interface change, this package will no longer compile and we will be on notice that it needs to be updated.

The appearance of the blank identifier in this construct indicates that the declaration exists only for the type checking, not to create a variable. Don't do this for every type that satisfies an interface, though. By convention, such declarations are only used when there are no static conversions already present in the code, which is a rare event.

##Embedding

Go does not provide the typical, type-driven notion of subclassing, but it does have the ability to “borrow” pieces of an implementation by embedding types within a struct or interface.

Interface embedding is very simple. We've mentioned the io.Reader and io.Writer interfaces before; here are their definitions.

    type Reader interface {
        Read(p []byte) (n int, err error)
    }

    type Writer interface {
        Write(p []byte) (n int, err error)
    }

The io package also exports several other interfaces that specify objects that can implement several such methods. For instance, there is io.ReadWriter, an interface containing both Read and Write. We could specify io.ReadWriter by listing the two methods explicitly, but it's easier and more evocative to embed the two interfaces to form the new one, like this:

    // ReadWriter is the interface that combines the Reader and Writer interfaces.
    type ReadWriter interface {
        Reader
        Writer
    }

This says just what it looks like: A ReadWriter can do what a Reader does and what a Writer does; it is a union of the embedded interfaces (which must be disjoint sets of methods). Only interfaces can be embedded within interfaces.

The same basic idea applies to structs, but with more far-reaching implications. The bufio package has two struct types, bufio.Reader and bufio.Writer, each of which of course implements the analogous interfaces from package io. And bufio also implements a buffered reader/writer, which it does by combining a reader and a writer into one struct using embedding: it lists the types within the struct but does not give them field names.

    // ReadWriter stores pointers to a Reader and a Writer.
    // It implements io.ReadWriter.
    type ReadWriter struct {
        *Reader  // *bufio.Reader
        *Writer  // *bufio.Writer
    }

The embedded elements are pointers to structs and of course must be initialized to point to valid structs before they can be used. The ReadWriter struct could be written as

    type ReadWriter struct {
        reader *Reader
        writer *Writer
    }

but then to promote the methods of the fields and to satisfy the io interfaces, we would also need to provide forwarding methods, like this:

    func (rw *ReadWriter) Read(p []byte) (n int, err error) {
        return rw.reader.Read(p)
    }

By embedding the structs directly, we avoid this bookkeeping. The methods of embedded types come along for free, which means that bufio.ReadWriter not only has the methods of bufio.Reader and bufio.Writer, it also satisfies all three interfaces: io.Reader, io.Writer, and io.ReadWriter.

There's an important way in which embedding differs from subclassing. When we embed a type, the methods of that type become methods of the outer type, but when they are invoked the receiver of the method is the inner type, not the outer one. In our example, when the Read method of a bufio.ReadWriter is invoked, it has exactly the same effect as the forwarding method written out above; the receiver is the reader field of the ReadWriter, not the ReadWriter itself.

Embedding can also be a simple convenience. This example shows an embedded field alongside a regular, named field.

    type Job struct {
        Command string
        *log.Logger
    }

The Job type now has the Log, Logf and other methods of *log.Logger. We could have given the Logger a field name, of course, but it's not necessary to do so. And now, once initialized, we can log to the Job:

    job.Log("starting now...")

The Logger is a regular field of the Job struct, so we can initialize it in the usual way inside the constructor for Job, like this,

    func NewJob(command string, logger *log.Logger) *Job {
        return &Job{command, logger}
    }

or with a composite literal,

    job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}

If we need to refer to an embedded field directly, the type name of the field, ignoring the package qualifier, serves as a field name, as it did in the Read method of our ReaderWriter struct. Here, if we needed to access the *log.Logger of a Job variable job, we would write job.Logger, which would be useful if we wanted to refine the methods of Logger.

    func (job *Job) Logf(format string, args ...interface{}) {
        job.Logger.Logf("%q: %s", job.Command, fmt.Sprintf(format, args...))
    }

Embedding types introduces the problem of name conflicts but the rules to resolve them are simple. First, a field or method X hides any other item X in a more deeply nested part of the type. If log.Logger contained a field or method called Command, the Command field of Job would dominate it.

Second, if the same name appears at the same nesting level, it is usually an error; it would be erroneous to embed log.Logger if the Job struct contained another field or method called Logger. However, if the duplicate name is never mentioned in the program outside the type definition, it is OK. This qualification provides some protection against changes made to types embedded from outside; there is no problem if a field is added that conflicts with another field in another subtype if neither field is ever used.

##<a name="concurrency"></a>Concurrency
###<a name="share-by-communicating"></a>以通訊實現資料共享

（譯註：concurrency 一譯「共時性」；由於這個概念本身較為抽象，又為避免和心理學上的同時性 synchronicity 混淆，以下保留原文名詞）

Concurrent programming 是個非常值得探討的議題，對此，Go 也有一些需要專門提出來討論的部份。

在 concurrent 環境中共享變數內容需要一些精妙的技巧來保證其正確性，使得 concurrency programming 在許多情況下並不容易。在 Go 的生態裡，我們鼓勵另外一種做法ㄧㄧ利用 channel 來傳遞需要共享的值，並避免讓多個 thread 頻繁地共享資源；無時無刻只讓單一 goroutine 有權存取特定變數，籍以避免資料競態 (data race)。為了宣揚此類理念，我們有個簡單的口號：

以溝通實現共享，而非以共享實現溝通。

當然，這樣的做法並非此類問題唯一或最簡單的解決方向，例如，在同步鎖 (mutex) 保護下的參照計數 (reference count) 一樣可以在多個 thread 之間維持資料正確性。但以高階程式語言的設計觀點，利用 channel 來控制資料分享是更淺顯且不易出錯的方案。

對於這種設計思維，這邊有一種簡單的理解方式。設想一支在單核心處理器上執行的典型的單一 thread 程式，以它的架構當然毋需考慮任何資料同步問題；現在把兩個這樣的架構並聯在一起，因為各別仍是獨立的單一 thread 架構，所以仍不需考慮同步。接著，我們讓這兩個 thread 進行通訊；如果通訊機制本身已經針對資料同步問題特別設計過，那麼整體而言仍不需對架構作出任何修改。UNIX pipe 正式此類設計的一種體現。雖然 Go 在這方面是基於 Hoare 提出的 Communicating Sequential Processes (CSP) 而設計，我們仍然可以把 channel 視為是一種型別安全的 UNIX pipe 延伸產物。

###<a name="goroutines"></a>Goroutines

之所以提出 *goroutine* 這個特殊名詞，是因為它並非單單只是傳統概念上的 thread、coroutine、甚至 process。一個 goroutine 只是一個單純的模型：一個函式與其它 goroutine 同時在同一個記憶體空間內執行。它非常輕量，用到的記憶體只比 call stack 多一點點；而 call stack 本身並不大，所以整體而言創造 goroutine 沒什麼成本，只隨著執行需求動態地增減所用到的 heap 空間。

多個 goroutine 會在多個作業系統 thread 上同時運作，所以假若某個 goroutine 因為等待系統 I/O 之類的理由而被阻塞，其它的 goroutine 會繼續執行不受其影響。其實作在台面下其實隱藏許多複雜的 thread 管理機制。

如果在函式調用時在前面加上 `go` 這個關鍵字，那麼這個調用就會被放到一個新的 goroutine 上執行。隨著該函式返回，goroutine 也會無聲無息地隨之結束。（這就好比 UNIX shell 裡面，如果在一行指令最後加上 `&` 符號，那麼這行指令會靜靜地在背景執行）

```go
go list.Sort()  // 非同步調用 list.Sort；我們不會在此等待其執行結果
```

匿名函式有時會讓我們能更方便地使用 goroutine。

```go
func Announce(message string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println(message)
    }()  // 注意這些圓括號ㄧㄧ這邊必須要進行函式調用
}
```

在 Go 的世界裡，匿名函式本身是一種閉包 (closure)：所有在此函式所能存取的變數壽命將持續到函式結束為止。

以上範例其實不太實用，因為我們無法得知函式何時結束。為了做到這點，我們需要 channel。

###<a name="channels"></a>Channels

一如 map，我們利用 `make` 創建 channel，且這些 channel 的行為與那些底層另有實體資料的參照型別相近（譯註：例如 slice 底層另有 array，可視為 array 的參照）。在利用 `make` 創建 channel 時有一個選填的整數參數，用以指定 channel 的緩衝區大小，預設為零，意指無緩衝的同步式 channel。

```go
ci := make(chan int)            // 無緩衝區的整數 channel
cj := make(chan int, 0)         // 同上，無緩衝區的整數 channel
cs := make(chan *os.File, 100)  // 有緩衝區的 channel，用以傳遞 *os.File
```

無緩衝的 channel 會把通訊內容，也就是用以傳遞的值，與同步機制結合在一起，如此便可保證兩個 goroutine 之間能在確定彼此的狀態的情況下交換資訊。

channel 能使得我們的程式更為簡潔優雅。讓我們回顧一下前面在背景計算排序的例子。如果在此使用 channel，則可籍此得知各分枝出去的 goroutine 執行狀態，進而在必要時等待其執行完成。

```go
c := make(chan int)  // 創建一個 channel
// 發起一個 goroutine，在其上進行排序計算。當計算完成時，透過 channel 通知我們。
go func() {
    list.Sort()
    c <- 1  // 發出結束訊號。其實際值為何無關緊要。
}()
doSomethingForAWhile()
<-c   // 等待排序完成；不理會實際收到的值。
```

channel 的接收端會在有東西能讓它接收之前會一直被阻塞。如果是無緩衝區的 channel，則發送端在有接收端就緒之前也會被卡住。如果 channel 有緩衝區，則發送端只須等待寄送內容被複製進緩衝區即可；但若緩衝區滿了，則發送端仍須等到有接收端先來把一些值收走。

一個有緩衝的 channel 可以當作號誌使用，例如用以作為流量控制的手段。在以下的例子裡，接收進來的請求都會被丟到 `handle` 裡處理，每個 `handle` 從 channel 裡面取值，處理請求內容，並在完成後把一個值塞回 channel，表示現在準備處理下一個請求。在此例中 channel 的緩衝區大小就相當於同時運行的 `process` 數量上限，因此在初始化這個 channel 的時候必須先把它的緩衝區填滿。

```go
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
    <-sem          // 等待取得執行權
    process(r)     // 處理請求。可能花上一點時間
    sem <- 1       // 處理完成；準備處理下一份請求
}

func init() {
    for i := 0; i < MaxOutstanding; i++ {
        sem <- 1
    }
}

func Serve(queue chan *Request) {
    for {
        req := <-queue
        go handle(req)  // 不在此等待 handle 執行完成
    }
}
```

在此，因為從 channel 取得資料才可能發生資料同步問題（意即，送出「在」接收「之前」；請參考《[The Go Memory Model](http://golang.org/ref/mem)》一文），所以必須在接收資料的時候取得執行權，而非在送出時取得。

前述設計仍有一個問題。雖然 `MaxOutstanding` 限制了同時運行的數量，但 `Serve` 每收到一個請求就會產生一個 goroutine；若請求發生的速度夠快，這個程式仍有可能無限制地消耗系統資源。

```go
func Serve(queue chan *Request) {
    for req := range queue {
        <-sem
        go func() {
            process(req) // 仍有漏洞。等等我們會解釋
            sem <- 1
        }()
    }
}
```

上面這個例子的問題在於，在 Go 的 `for` 迴圈中，迭代變數在每輪迭代中被複用，所以 `req` 變數被每個 goroutine 所共享。我們必須在此保證每個 goroutine 使用各別的 `req`，因此下例中便把 `req` 作為參數帶入 closure 裡面：

```go
func Serve(queue chan *Request) {
    for req := range queue {
        <-sem
        go func(req *Request) {
            process(req)
            sem <- 1
        }(req)
    }
}
```

請比較一下以上幾個版本中 closure 的宣告與執行方式的差異；以下是另一種解決方式，此例子在每次迭代裡創建同名但全新的變數：

```go
func Serve(queue chan *Request) {
    for req := range queue {
        <-sem
        req := req // 為每一個 goroutine 創建全新的 req
        go func() {
            process(req)
            sem <- 1
        }()
    }
}
```

這種寫法可能看起來很奇怪：

```go
req := req
```

但這在 Go 是很常見的做法。我們用相同的名稱宣告全新的變數，籍此屏壁迭代時共用的變數，使得每個 goroutine 得以使用獨立的區域變數而非共享之。

讓我們回到最前面的請求處理問題。另一種可行的設計架構是直接固定 `handle` 的 goroutine 數量，限制了 goroutine 數量等同於限制了 `process` 的同時執行數。`Serve` 函式透過一個 channel 得知自己什麼時候該結束執行，而自 goroutine 開始執行之後，這個 channel 就會一直被卡住，直到有東西傳遞為止。

```go
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // 啟動 handle 處理資料
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // 持續等待，直到收到東西為止
}
```

###<a name="channels-of-channels"></a>用 channel 傳遞 channel

Channel 在 Go 裡面是主要資料型別之一 (first-class value)。這點相當重要，意味著 channel 也可以和其它型態的值一樣被分配和傳遞。這樣的特性常被用來實現安全的平行化分工機制。

在前一節提到的各種案例裡，`handle` 是一個理想化的請求處理元件，但我們並沒有深入定義所謂「請求」的實際資料結構。我們可以在其內附上一個 channel，讓各個終端得以籍其回報處理結果。下面這個例子示範了 `Request` 可能的定義：

```go
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}
```

籍此，請求者在 `Request` 中填入想要執行的函式、參數，以及用以接收結果的 channel。

```go
func sum(a []int) (s int) {
    for _, v := range a {
        s += v
    }
    return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// 送出請求
clientRequests <- request
// 等待回報結果
fmt.Printf("answer: %d\n", <-request.resultChan)
```

在伺服端，我們只需要把 handle 的函式內容修改一下：

```go
func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```

當然，在前面我們討論的案例中，仍有許多細節需要加以調整，但大體上已經是一個堪用的程式架構，兼具運算資源控制、平行化、非阻塞式通訊等特質，且並未（顯式地）使用任何同步鎖。

###<a name="parallelization"></a>平行處理 (Parallelization)

另一種實現 concurrency 的方法，則是直接利用多核心處理器進行平行運算。如果整個運算過程可以拆解成多個獨立的片斷，則可將運算平行化，並利用 channel 來串連各個片段。

舉例來說，假設現在我們有個理想化的模型，模型中有一串資料，每筆資料運算成本昂貴但彼此獨立：

```go
type Vector []float64

// 把每個元素 v[i], v[i+1] ... v[n-1] 分別代入運算
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
    for ; i < n; i++ {
        v[i] += u.Op(v[i])
    }
    c <- 1    // 回報這部份已完成計算
}
```

我們把待處理資料等分，在各個處理器實體上獨立進行計算；因此運算順序並不固定，但在此我們不關心次序問題。籍由統計從 channel 來的訊號，我們可知目前各片段的完成進度。

```go
const NCPU = 4  // 處理器總數

func (v Vector) DoAll(u Vector) {
    c := make(chan int, NCPU)  // 雖然緩衝區大小可設可不設，但此例中有其意義
    for i := 0; i < NCPU; i++ {
        go v.DoSome(i*len(v)/NCPU, (i+1)*len(v)/NCPU, u, c)
    }
    // 不斷從 channel 中汲取訊號
    for i := 0; i < NCPU; i++ {
        <-c    // 等待各片斷完成運算
    }
    // 全部完成
}
```

以目前 Go 的實作，上例中的各 goroutine 並非真正物理上平行運作。在預設環境下的每個時間點，任意數量的 goroutine 可以同時進行系統調用 (system call)、並阻塞到調用完成，但只能有一個 goroutine 能真正在作業系統的使用者模式 (user-level) 下運行。目前這種單一 thread 的設計有改善的空間，未來會實作成真正意義的平行化；但若我們想以目前的實作環境中享受多處理器帶來的好處，則必須把 goroutine 的並行數量上限明確地告訴執行環境。現階段有兩種設定方式：一種是在執行程式之前，把處理器核心數量設在 `GOMAXPROCS` 這個環境變數裡；另一種方式，則是引入 `runtime` 這個 package，並調用 `rutime.GOMAXPROCS(NCPU)` 設定之。`runtime.NumCPU()` 可以查詢當前硬體平台的邏輯處理器個數，必要時可以加以利用。再次提醒，以上設定只是目前的權宜之計，一旦將來 goroutine 的排程相關實作得到改善，這些平行化環境的初始化設定都可以不用刻意執行。

最後再度強調，請勿將 concurrency 與 parallelism 混為一談ㄧㄧ前者將整個計算結構拆解成各自獨立的片段，後者則是同時在多個處理器上進行運算。即使 Go 針對 concurrency 的設計可能有利於處理某些平行運算問題，Go 在本質上仍只是針對 concurrency 而設計，而非 parallelism，因此 Go 並非適合所有的平行運算問題。[這篇文章](http://blog.golang.org/2013/01/concurrency-is-not-parallelism.html)深入討論了「concurrency」與「parallelism」之間的差異。

###<a name="a-leaky-buffer"></a>不會滿溢的緩衝區

Concurrency programming 的技巧有時更能表現某些非 concurrent 的思維。以下這個例子來自於某個抽象化之後的 RPC package。用戶端 goroutine 不斷從某個來源（也許是網路）接收資料。為了避免頻繁地創建／銷毀緩衝區而對效能造成不利的影響，在此我們用一個帶有緩衝的 channel 作為一組鏈表，用以儲存沒有在使用的備用緩衝空間；每當需要新的緩衝區，直接從這裡取用；當這個 channel 空了，才真正向系統請求一塊記憶體。無論是先前儲存的、或是剛從系統拿到的，一旦準備好新緩衝區，就把它塞進 `serverChan`。

```go
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
    for {
        var b *Buffer
        // 試著從快取裡面拿一個用過的緩衝區；如果拿不到，那就做一個新的
        select {
        case b = <-freeList:
            // 拿一個舊的出來，除此之外沒有額外動作
        default:
            // 沒有空閒的緩衝區可用，只好弄一個新的來
            b = new(Buffer)
        }
        load(b)              // 從網路上讀一段資料進來
        serverChan <- b      // 把資料往伺服端遞送
    }
}
```

伺服端持續從用戶端接收資料，處理資料，然後歸還緩衝區。

```go
func server() {
    for {
        b := <-serverChan    // 等待新工作上門
        process(b)
        // 如果快取還有空位的話，就把緩衝區還到快取裡面
        select {
        case freeList <- b:
            // 把連西塞回鏈表；除此之外不做事
        default:
            // 鏈表滿了，直接繼續往下執行
        }
    }
}
```

用戶端試圖從 `freeList` 取緩衝區來用；如果拿不到，則創造一個全新的出來。伺服端每次用完之後就把 `b` 塞回緩衝區快取鏈表 `freeList`；如果鏈表滿了，就直接不多作處理，把過剩的緩衝扔在路邊，過陣子會自動被垃圾回收機制處理掉（在 `select` 中的 `default` 敘述句只會在沒有任何的 `case` 項目可以馬上執行時才會進入；也就是說，帶有 `default` 的 `select` 區塊是不會被阻塞的）。這短短數行只利用緩衝 channel 和垃圾回收機制，就實現了自動控制容量、不會滿溢的快取鏈表。

##<a name="errors"></a>錯誤處理

函式庫提供各式各樣的工具元件，有時也必須回報適當的錯誤訊息給調用者。如同先前提過，Go 允許函式一次返回多個值；利用這點，我們可以在返迴一般數值結果的同時描述錯誤的發生狀況。慣例上，錯誤訊息會包裝成 `error`，這是一種內建的 interface。

```go
type error interface {
    Error() string
}
```

函式庫開發者可以自由地實現這個 interface，闊展其內容，甚至在描述問題的同時也把執行期上下文包裝進去，`os.Open` 所返回的 `os.PathError` 便是一例。

```go
// PathError 不但描述錯誤訊息，
// 同時也記載當時對哪個檔案進行什麼操作
type PathError struct {
    Op string    // 諸如 "open" 或 "unlink" …等等
    Path string  // 欲操作的檔案路徑
    Err error    // 系統呼叫產生的錯誤代碼
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

`PathError` 的 `Error` 介面在發生問題可能會產生類似如下描述：

    open /etc/passwx: no such file or directory
    （譯註：open /etc/passwx: 找不到檔案或資料夾）

這段敘述指出哪個檔案發生問題、當時做了什麼事、以及作業系統回報錯誤的理由；即便我們不在事發當時馬上處理這個錯誤，事後仍有足夠清楚的資訊能瞭解當時發生了什麼事。這樣的錯誤訊息至少比單純抱怨「no such file or directory（找不到檔案或資料夾）」來得有意義得多。

錯誤訊息應儘可能地描述其發生原因，包括當時意圖進行的操作，還有這個錯誤來自於哪個 package。舉例而言，如果要利用 `image` 這個 package 裡面的方法對圖像資料進行解碼，但它不認得目標的實際編碼格式，則可能會得到像是「image: unknown format」這樣的訊息。

如果函式調用方想從 `error` 中得到更詳細的資訊，可以試著利用 type switch 或 type assertion 得到 error 的實際型態，進而從中解出細節。以這裡遇到的 `PathError` 為例，籍由比對 `Err` 欄位，我們得以在問題發生後採取補救手段。

```go
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        deleteTempFiles()  // 釋放一些不用的空間
        continue
    }
    return
}
```

上例中的第二個 if 判斷式裡面是一條 type assertion。如果轉型失敗，`ok` 的值會變成 `false`，而 `e` 會變成 `nil`；反之若轉型成功，`ok` 則會變成 `true`，意即 `error` 實際上可以還原成 `*os.PathError`，我們得以從還原後的 `e` 查出更多的相關資訊。

###<a name="panic"></a>Panic

一般常見的錯誤回報機制，是把 `error` 作為多重返回值之一傳回去。標準函式庫中的 `Read` 是一個眾所皆知的典型範例，它除了返回實際讀入的 byte 數，同時附帶一個 `error`。但，萬一發生的意外其實根本無法作出任何補救呢？有時候程式在這種情況下根本無法繼續運作。

為了應付這種狀況，我們可以利用內建函式 `panic` 產生一個執行期錯誤（run-time error），這會使得程式直接停止（這樣的說法其實不夠完整，我們在下一節會解釋）。這個函式接受一個任意種類的參數，作為程式死亡時的線索，習慣上會在此傳入說明用的字串。我們也可以利用這個函式指出邏輯上沒有道理的錯誤，例如跳離一個不該跳離的無限迴圈。

```go
// 利用牛頓法求立方根的小玩具
func CubeRoot(x float64) float64 {
    z := x/3   // 給定任意的初始值
    for i := 0; i < 1e6; i++ {
        prevz := z
        z -= (z*z*z-x) / (3*z*z)
        if veryClose(z, prevz) {
            return z
        }
    }
    // 經過一百萬次迭代仍未收斂，一定有哪邊出了問題
    panic(fmt.Sprintf("CubeRoot(%g) 沒有收斂", x))
}
```

這只是一個小小的範例，但實際應用上我們應該避免在設計函式庫時使用 `panic`。如果一個問題可以用其它方式掩蓋或補救，對整支程式而言依然是好死不如賴活。以下是少數的反例，在程式初始化時使用 `panic`：如果一個函式庫連最基本的初始準備都無法順利完成，那麼也只有 `panic` 一途。

```go
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("找不到 $USER 這個值")
    }
}
```

###<a name="recove"></a>Recover

當 `panic` 發生，無論其肇因於某次存取 slice 時超過其合法長度、或是 type assertion 失敗等等造成的執行期錯誤，正在執行的函式都會馬上中斷，接著會從當前所在位置開始、把先前的 deferred function 一一展開執行。如果最後展開到其 goroutine 的 call stack 頂端，整支程式就往生了。然而，內建函式 `recover` 可以讓我們在發生 panic 時重掌 goroutine，進而使程式回歸正軌。

調用 `recover` 可以阻止繼續展開 deferred functon，並回傳把先前塞進 panic 的參數。因為 panic 情況下的展開過程只會執行 deferred function 內的邏輯，所以 recover 只在被 deferred function 內才派得上用場。

`recover` 的其中一種應用，是在伺服器內用以善後執行失敗的 goroutine，防止其中某個 goroutine 發生問題的同時一併抹殺其它 goroutine。

```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("處理失敗:", err)
        }
    }()
    do(work)
}
```

上例中的 `do(work)` 一旦發生 panic，則這個事件會被記錄下來，完成善後之後 goroutine 自己安靜地結束，不致影響其它 goroutine。整個 deferred closure 內容就是如此單純，僅須調用 `recover` 並處理其結果。

因為在 deferred function 以外的情況使用 `recover` 只會得到 `nil`，所以 deferred function 的內部仍可正常使用任何函式，即便它可能 panic。以上例來說，`safelyDo` 內的 deferred function 裡，在調用 `recover` 之前仍可執行記錄事件的相關邏輯，log package 內的元件不會因為目前所處的環境是 panic stack 而受到影響。

在剛剛提出的錯誤回復架構裡，`do` 函式（與其內調用的其它函式們）可以籍由調用 `panic` 輕易地跳離任何困境。這種手法得以簡化整個錯誤處理機制，在開發大型程式時尤其實用。讓我們看看下面這個例子，這是一個理想化的正規表示式函式庫，它會在語法無法正確解析的時候產生一個自定義的 panic。以下片段包括 `Error` 型別的定義、型別方法 `error`，以及 `Compile` 函式的內容：

```go
// Error 型別用以表示語法錯誤；這個型別實作了 error 介面。
type Error string
func (e Error) Error() string {
    return string(e)
}

// 當 *Regexp 試圖回報語法解析錯誤，會籍由 panic 把 Error 報出來，
// 而 error 則是這個 Error 用以敘述問題的方法
func (regexp *Regexp) error(err string) {
    panic(Error(err))
}

// Compile 會傳回正規表示式的解析結果
func Compile(str string) (regexp *Regexp, err error) {
    regexp = new(Regexp)
    // 如果語法解析出了問題，doParse 會發生 panic
    defer func() {
        if e := recover(); e != nil {
            regexp = nil    // 清空返回值
            err = e.(Error) // 如果轉型失敗，則會發生新的 panic
        }
    }()
    return regexp.doParse(str), nil
}
```

如果 `doParse` 發生 panic，這裡會將返回值回復成 `nil` ㄧㄧ deferred function 內可以直接存取這個返回變數。接著請看 `err` 賦值這行，在此確認 panic 扔出值的實際型別是否的確是前面自定義的 `Error`；如果不是，則 type assertion 失敗，此時產生執行期錯誤，繼續把 panic stack 的各函式向上展開。後面這個段檢驗意味著，如果發生了什麼真正超出預期的事（例如記憶體存取越界），這段程式碼不會因為有用到 `recover` 來處理 `panic` 就一定能正常結束。

`Error` 的 `error` 結繫於資料型別之上，故稱為方法；其與內建型別 `error` 同名，這沒什麼問題，而且渾然天成。有了以上的錯誤處理設計，使人得以更方便地回報語法錯問題，而不需要在發生問題時自己設法展開整個解析進度。

```go
if pos == 0 {
    re.error("符號 '*' 不得置於句首")
}
```

雖然這種設計很實用，但這該被限制只在 package 內部使用。`Parse` 把內部 `panic` 的內容當作是一般的 `error` 返回值，而不將 panic 細節暴露給 package 使用者。這是一個值得遵循的規則。

順帶一提，這種重覆 panic 的設計模式會覆寫原有的 panic 內容。然而無論新舊錯誤訊息，都會在程式崩壞時回報出來，因此問題的根源依然有跡可循。以上提到的這個重覆 panic 手法總是相當實用，雖然最終仍會導致程式崩壞，但讓人得以在處理 panic 時插入一些內容以觀察變數實際內容，過濾不想理會的部份，最後再度把原值 panic 出去。至於解決問題的工作，則留給看到這些錯誤訊息的人。

##A web server

 Let's finish with a complete Go program, a web server. This one is actually a kind of web re-server. Google provides a service at http://chart.apis.google.com that does automatic formatting of data into charts and graphs. It's hard to use interactively, though, because you need to put the data into the URL as a query. The program here provides a nicer interface to one form of data: given a short piece of text, it calls on the chart server to produce a QR code, a matrix of boxes that encode the text. That image can be grabbed with your cell phone's camera and interpreted as, for instance, a URL, saving you typing the URL into the phone's tiny keyboard.

Here's the complete program. An explanation follows.

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
    <form action="/" name=f method="GET"><input maxLength=1024 size=70
    name=s value="" title="Text to QR Encode"><input type=submit
    value="Show QR" name=qr>
    </form>
    </body>
    </html>
`

The pieces up to main should be easy to follow. The one flag sets a default HTTP port for our server. The template variable templ is where the fun happens. It builds an HTML template that will be executed by the server to display the page; more about that in a moment.

The main function parses the flags and, using the mechanism we talked about above, binds the function QR to the root path for the server. Then http.ListenAndServe is called to start the server; it blocks while the server runs.

QR just receives the request, which contains form data, and executes the template on the data in the form value named s.

The template package html/template is powerful; this program just touches on its capabilities. In essence, it rewrites a piece of HTML text on the fly by substituting elements derived from data items passed to templ.Execute, in this case the form value. Within the template text (templateStr), double-brace-delimited pieces denote template actions. The piece from {{if .}} to {{end}} executes only if the value of the current data item, called . (dot), is non-empty. That is, when the string is empty, this piece of the template is suppressed.

The two snippets {{.}} say to show the data presented to the template—the query string—on the web page. The HTML template package automatically provides appropriate escaping so the text is safe to display.

The rest of the template string is just the HTML to show when the page loads. If this is too quick an explanation, see the documentation for the template package for a more thorough discussion.

And there you have it: a useful web server in a few lines of code plus some data-driven HTML text. Go is powerful enough to make a lot happen in a few lines.

---

Build version go1.1.2.
Except as noted, the content of this page is licensed under the Creative Commons Attribution 3.0 License, and code is licensed under a BSD license.
