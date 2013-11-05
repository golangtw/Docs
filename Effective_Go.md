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
- [Data](#data)
    - [Allocation with new](#allocation-with-new)
    - [Constructors and composite literals](#constructors-and-composite-literals)
    - [Allocation with make](#allocation-with-make)
    - [Arrays](#arrays)
    - [Slices](#slices)
    - [Two-dimensional slices](#two-dimensional-slices)
    - [Maps](#maps)
    - [Printing](#printing)
    - [Append](#append)
- [Initialization](#initialization)
    - [Constants](#constants)
    - [Variables](#variables)
    - [The init function](#the-init-function)
- [Methods](#methods)
    - [Pointers vs. Values](#pointers-vs-values)
- [Interfaces and other types](#interfaces-and-other-types)
    - [Interfaces](#interfaces)
    - [Conversions](#conversions)
    - [Interface conversions and type assertions](#interface-conversions-and-type-assertions)
    - [Generality](generality)
    - [Interfaces and methods](#interfaces-and-methods)
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

格式的問題最容易引起爭論同時也是最不重要的。人們可以適應各種不同的格式，但最好的情況是他們不需要這麼做，如果每個人都遵循相同的格式，就可以花更少的時間在這個議題上。問題在於要如何不透過冗長的規範手冊達到這樣的烏托邦境界。

在Go中我們採用了一種不尋常的方法 - 讓機器處理大部分的格式問題。gofmt指令(也可以使用go fmt，它是在package層級運作而非source層級)可以讀取一份go程式並產出以標準格式縮排以及垂直排版的原始碼，保留注釋的格式，但有必要的話會重排。如果你想要知道如何處理一些新的編排狀況，執行gofmt;如果結果看起來不太對，重新整理你的程式(或是提交關於gofmt的bug)，不要使用能避開問題的替代方法。

如同範例所示，不需要花時間對齊struct欄位後面的注釋。Gofmt會幫你處理。宣告一個struct
```go
    type T struct {
        name string // 物件的名字
        value int // 它的值
    }
```
gofmt將會對齊欄位
```go
    type T struct {
        name    string // 物件的名字
        value   int    // 它的值
    }
```
所有Go標準函式庫中的程式碼都使用了gofmt格式化。

一些格式的細節仍然存在。簡單的說：

縮進
    我們使用tabs做為縮進而且讓gofmt預設產出它。當你真正需要時才使用空白。

行長
    Go沒有行的長度限制。不用擔心超過穿孔卡片。如果覺得某行太長，換行並且使用額外的縮進。

圓括號
    比起C與Java，Go需要的圓括號更少：控制結構(if, for, switch)在它們的語法中不需要圓括號。同樣的，運算子優先層級可以變得更短更清楚，所以
```go 
    x<<8 + y<<16
```
表示出間隔所暗示的意義，不像在其他語言裡一樣隱晦。

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

##Data
###Allocation with new

Go has two allocation primitives, the built-in functions new and make. They do different things and apply to different types, which can be confusing, but the rules are simple. Let's talk about new first. It's a built-in function that allocates memory, but unlike its namesakes in some other languages it does not initialize the memory, it only zeros it. That is, new(T) allocates zeroed storage for a new item of type T and returns its address, a value of type *T. In Go terminology, it returns a pointer to a newly allocated zero value of type T.

Since the memory returned by new is zeroed, it's helpful to arrange when designing your data structures that the zero value of each type can be used without further initialization. This means a user of the data structure can create one with new and get right to work. For example, the documentation for bytes.Buffer states that "the zero value for Buffer is an empty buffer ready to use." Similarly, sync.Mutex does not have an explicit constructor or Init method. Instead, the zero value for a sync.Mutex is defined to be an unlocked mutex.

The zero-value-is-useful property works transitively. Consider this type declaration.

    type SyncedBuffer struct {
        lock    sync.Mutex
        buffer  bytes.Buffer
    }

Values of type SyncedBuffer are also ready to use immediately upon allocation or just declaration. In the next snippet, both p and v will work correctly without further arrangement.

    p := new(SyncedBuffer)  // type *SyncedBuffer
    var v SyncedBuffer      // type  SyncedBuffer

###Constructors and composite literals

Sometimes the zero value isn't good enough and an initializing constructor is necessary, as in this example derived from package os.

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

There's a lot of boiler plate in there. We can simplify it using a composite literal, which is an expression that creates a new instance each time it is evaluated.

    func NewFile(fd int, name string) *File {
        if fd < 0 {
            return nil
        }
        f := File{fd, name, nil, 0}
        return &f
    }

Note that, unlike in C, it's perfectly OK to return the address of a local variable; the storage associated with the variable survives after the function returns. In fact, taking the address of a composite literal allocates a fresh instance each time it is evaluated, so we can combine these last two lines.

    return &File{fd, name, nil, 0}

The fields of a composite literal are laid out in order and must all be present. However, by labeling the elements explicitly as field:value pairs, the initializers can appear in any order, with the missing ones left as their respective zero values. Thus we could say

    return &File{fd: fd, name: name}

As a limiting case, if a composite literal contains no fields at all, it creates a zero value for the type. The expressions new(File) and &File{} are equivalent.

Composite literals can also be created for arrays, slices, and maps, with the field labels being indices or map keys as appropriate. In these examples, the initializations work regardless of the values of Enone, Eio, and Einval, as long as they are distinct.

    a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
    s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
    m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}

###Allocation with make

Back to allocation. The built-in function make(T, args) serves a purpose different from new(T). It creates slices, maps, and channels only, and it returns an initialized (not zeroed) value of type T (not *T). The reason for the distinction is that these three types represent, under the covers, references to data structures that must be initialized before use. A slice, for example, is a three-item descriptor containing a pointer to the data (inside an array), the length, and the capacity, and until those items are initialized, the slice is nil. For slices, maps, and channels, make initializes the internal data structure and prepares the value for use. For instance,

    make([]int, 10, 100)

allocates an array of 100 ints and then creates a slice structure with length 10 and a capacity of 100 pointing at the first 10 elements of the array. (When making a slice, the capacity can be omitted; see the section on slices for more information.) In contrast, new([]int) returns a pointer to a newly allocated, zeroed slice structure, that is, a pointer to a nil slice value.

These examples illustrate the difference between new and make.

    var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
    var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

    // Unnecessarily complex:
    var p *[]int = new([]int)
    *p = make([]int, 100, 100)

    // Idiomatic:
    v := make([]int, 100)

Remember that make applies only to maps, slices and channels and does not return a pointer. To obtain an explicit pointer allocate with new or take the address of a variable explicitly.

###Arrays

Arrays are useful when planning the detailed layout of memory and sometimes can help avoid allocation, but primarily they are a building block for slices, the subject of the next section. To lay the foundation for that topic, here are a few words about arrays.

There are major differences between the ways arrays work in Go and C. In Go,

- Arrays are values. Assigning one array to another copies all the elements.
- In particular, if you pass an array to a function, it will receive a copy of the array, not a pointer to it.
- The size of an array is part of its type. The types [10]int and [20]int are distinct.

The value property can be useful but also expensive; if you want C-like behavior and efficiency, you can pass a pointer to the array.

    func Sum(a *[3]float64) (sum float64) {
        for _, v := range *a {
            sum += v
        }
        return
    }

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)  // Note the explicit address-of operator

But even this style isn't idiomatic Go. Use slices instead.

###Slices

Slices wrap arrays to give a more general, powerful, and convenient interface to sequences of data. Except for items with explicit dimension such as transformation matrices, most array programming in Go is done with slices rather than simple arrays.

Slices hold references to an underlying array, and if you assign one slice to another, both refer to the same array. If a function takes a slice argument, changes it makes to the elements of the slice will be visible to the caller, analogous to passing a pointer to the underlying array. A Read function can therefore accept a slice argument rather than a pointer and a count; the length within the slice sets an upper limit of how much data to read. Here is the signature of the Read method of the File type in package os:

    func (file *File) Read(buf []byte) (n int, err error)

The method returns the number of bytes read and an error value, if any. To read into the first 32 bytes of a larger buffer b, slice (here used as a verb) the buffer.

        n, err := f.Read(buf[0:32])

Such slicing is common and efficient. In fact, leaving efficiency aside for the moment, the following snippet would also read the first 32 bytes of the buffer.

    var n int
    var err error
    for i := 0; i < 32; i++ {
        nbytes, e := f.Read(buf[i:i+1])  // Read one byte.
        if nbytes == 0 || e != nil {
            err = e
            break
        }
        n += nbytes
    }

The length of a slice may be changed as long as it still fits within the limits of the underlying array; just assign it to a slice of itself. The capacity of a slice, accessible by the built-in function cap, reports the maximum length the slice may assume. Here is a function to append data to a slice. If the data exceeds the capacity, the slice is reallocated. The resulting slice is returned. The function uses the fact that len and cap are legal when applied to the nil slice, and return 0.

    func Append(slice, data[]byte) []byte {
        l := len(slice)
        if l + len(data) > cap(slice) {  // reallocate
            // Allocate double what's needed, for future growth.
            newSlice := make([]byte, (l+len(data))*2)
            // The copy function is predeclared and works for any slice type.
            copy(newSlice, slice)
            slice = newSlice
        }
        slice = slice[0:l+len(data)]
        for i, c := range data {
            slice[l+i] = c
        }
        return slice
    }

We must return the slice afterwards because, although Append can modify the elements of slice, the slice itself (the run-time data structure holding the pointer, length, and capacity) is passed by value.

The idea of appending to a slice is so useful it's captured by the append built-in function. To understand that function's design, though, we need a little more information, so we'll return to it later.

###Two-dimensional slices

Go's arrays and slices are one-dimensional. To create the equivalent of a 2D array or slice, it is necessary to define an array-of-arrays or slice-of-slices, like this:

    type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
    type LinesOfText [][]byte     // A slice of byte slices.

Because slices are variable-length, it is possible to have each inner slice be a different length. That can be a common situation, as in our LinesOfText example: each line has an independent length.

    text := LinesOfText{
        []byte("Now is the time"),
        []byte("for all good gophers"),
        []byte("to bring some fun to the party."),
    }

Sometimes it's necessary to allocate a 2D slice, a situation that can arise when processing scan lines of pixels, for instance. There are two ways to achieve this. One is to allocate each slice independently; the other is to allocate a single array and point the individual slices into it. Which to use depends on your application. If the slices might grow or shrink, they should be allocated independently to avoid overwriting the next line; if not, it can be more efficient to construct the object with a single allocation. For reference, here are sketches of the two methods. First, a line a time:

    // Allocate the top-level slice.
    picture := make([][]uint8, YSize) // One row per unit of y.
    // Loop over the rows, allocating the slice for each row.
    for i := range picture {
        picture[i] = make([]uint8, XSize)
    }

And now as one allocation, sliced into lines:

    // Allocate the top-level slice, the same as before.
    picture := make([][]uint8, YSize) // One row per unit of y.
    // Allocate one large slice to hold all the pixels.
    pixels := make([]uint8, XSize*YSize) // Has type []uint8 even though picture is [][]uint8.
    // Loop over the rows, slicing each row from the front of the remaining pixels slice.
    for i := range picture {
        picture[i], pixels = pixels[:XSize], pixels[XSize:]
    }

###Maps

Maps are a convenient and powerful built-in data structure that associate values of one type (the key) with values of another type (the element or value) The key can be of any type for which the equality operator is defined, such as integers, floating point and complex numbers, strings, pointers, interfaces (as long as the dynamic type supports equality), structs and arrays. Slices cannot be used as map keys, because equality is not defined on them. Like slices, maps hold references to an underlying data structure. If you pass a map to a function that changes the contents of the map, the changes will be visible in the caller.

Maps can be constructed using the usual composite literal syntax with colon-separated key-value pairs, so it's easy to build them during initialization.

    var timeZone = map[string]int{
        "UTC":  0*60*60,
        "EST": -5*60*60,
        "CST": -6*60*60,
        "MST": -7*60*60,
        "PST": -8*60*60,
    }

Assigning and fetching map values looks syntactically just like doing the same for arrays and slices except that the index doesn't need to be an integer.

    offset := timeZone["EST"]

An attempt to fetch a map value with a key that is not present in the map will return the zero value for the type of the entries in the map. For instance, if the map contains integers, looking up a non-existent key will return 0. A set can be implemented as a map with value type bool. Set the map entry to true to put the value in the set, and then test it by simple indexing.

    attended := map[string]bool{
        "Ann": true,
        "Joe": true,
        ...
    }

    if attended[person] { // will be false if person is not in the map
        fmt.Println(person, "was at the meeting")
    }

Sometimes you need to distinguish a missing entry from a zero value. Is there an entry for "UTC" or is that the empty string because it's not in the map at all? You can discriminate with a form of multiple assignment.

    var seconds int
    var ok bool
    seconds, ok = timeZone[tz]

For obvious reasons this is called the “comma ok” idiom. In this example, if tz is present, seconds will be set appropriately and ok will be true; if not, seconds will be set to zero and ok will be false. Here's a function that puts it together with a nice error report:

    func offset(tz string) int {
        if seconds, ok := timeZone[tz]; ok {
            return seconds
        }
        log.Println("unknown time zone:", tz)
        return 0
    }

To test for presence in the map without worrying about the actual value, you can use the blank identifier (_) in place of the usual variable for the value.

    _, present := timeZone[tz]

To delete a map entry, use the delete built-in function, whose arguments are the map and the key to be deleted. It's safe to do this even if the key is already absent from the map.

    delete(timeZone, "PDT")  // Now on Standard Time

###Printing

Formatted printing in Go uses a style similar to C's printf family but is richer and more general. The functions live in the fmt package and have capitalized names: fmt.Printf, fmt.Fprintf, fmt.Sprintf and so on. The string functions (Sprintf etc.) return a string rather than filling in a provided buffer.

You don't need to provide a format string. For each of Printf, Fprintf and Sprintf there is another pair of functions, for instance Print and Println. These functions do not take a format string but instead generate a default format for each argument. The Println versions also insert a blank between arguments and append a newline to the output while the Print versions add blanks only if the operand on neither side is a string. In this example each line produces the same output.

    fmt.Printf("Hello %d\n", 23)
    fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
    fmt.Println("Hello", 23)
    fmt.Println(fmt.Sprint("Hello ", 23))

The formatted print functions fmt.Fprint and friends take as a first argument any object that implements the io.Writer interface; the variables os.Stdout and os.Stderr are familiar instances.

Here things start to diverge from C. First, the numeric formats such as %d do not take flags for signedness or size; instead, the printing routines use the type of the argument to decide these properties.

    var x uint64 = 1<<64 - 1
    fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))

prints

    18446744073709551615 ffffffffffffffff; -1 -1

If you just want the default conversion, such as decimal for integers, you can use the catchall format %v (for “value”); the result is exactly what Print and Println would produce. Moreover, that format can print any value, even arrays, slices, structs, and maps. Here is a print statement for the time zone map defined in the previous section.

fmt.Printf("%v\n", timeZone)  // or just fmt.Println(timeZone)

which gives output

    map[CST:-21600 PST:-28800 EST:-18000 UTC:0 MST:-25200]

For maps the keys may be output in any order, of course. When printing a struct, the modified format %+v annotates the fields of the structure with their names, and for any value the alternate format %#v prints the value in full Go syntax.

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

prints

    &{7 -2.35 abc   def}
    &{a:7 b:-2.35 c:abc     def}
    &main.T{a:7, b:-2.35, c:"abc\tdef"}
    map[string] int{"CST":-21600, "PST":-28800, "EST":-18000, "UTC":0, "MST":-25200}

(Note the ampersands.) That quoted string format is also available through %q when applied to a value of type string or []byte. The alternate format %#q will use backquotes instead if possible. (The %q format also applies to integers and runes, producing a single-quoted rune constant.) Also, %x works on strings, byte arrays and byte slices as well as on integers, generating a long hexadecimal string, and with a space in the format (% x) it puts spaces between the bytes.

Another handy format is %T, which prints the type of a value.

    fmt.Printf("%T\n", timeZone)

prints

    map[string] int

If you want to control the default format for a custom type, all that's required is to define a method with the signature String() string on the type. For our simple type T, that might look like this.

    func (t *T) String() string {
        return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
    }
    fmt.Printf("%v\n", t)

to print in the format

    7/-2.35/"abc\tdef"

(If you need to print values of type T as well as pointers to T, the receiver for String must be of value type; this example used a pointer because that's more efficient and idiomatic for struct types. See the section below on pointers vs. value receivers for more information.)

Our String method is able to call Sprintf because the print routines are fully reentrant and can be wrapped this way. There is one important detail to understand about this approach, however: don't construct a String method by calling Sprintf in a way that will recur into your String method indefinitely. This can happen if the Sprintf call attempts to print the receiver directly as a string, which in turn will invoke the method again. It's a common and easy mistake to make, as this example shows.

    type MyString string

    func (m MyString) String() string {
        return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.
    }

It's also easy to fix: convert the argument to the basic string type, which does not have the method.

    type MyString string
    func (m MyString) String() string {
        return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
    }

In the initialization section we'll see another technique that avoids this recursion.

Another printing technique is to pass a print routine's arguments directly to another such routine. The signature of Printf uses the type ...interface{} for its final argument to specify that an arbitrary number of parameters (of arbitrary type) can appear after the format.

    func Printf(format string, v ...interface{}) (n int, err error) {

Within the function Printf, v acts like a variable of type []interface{} but if it is passed to another variadic function, it acts like a regular list of arguments. Here is the implementation of the function log.Println we used above. It passes its arguments directly to fmt.Sprintln for the actual formatting.

    // Println prints to the standard logger in the manner of fmt.Println.
    func Println(v ...interface{}) {
        std.Output(2, fmt.Sprintln(v...))  // Output takes parameters (int, string)
    }

We write ... after v in the nested call to Sprintln to tell the compiler to treat v as a list of arguments; otherwise it would just pass v as a single slice argument.

There's even more to printing than we've covered here. See the godoc documentation for package fmt for the details.

By the way, a ... parameter can be of a specific type, for instance ...int for a min function that chooses the least of a list of integers:

    func Min(a ...int) int {
        min := int(^uint(0) >> 1)  // largest int
        for _, i := range a {
            if i < min {
                min = i
            }
        }
        return min
    }

###Append

Now we have the missing piece we needed to explain the design of the append built-in function. The signature of append is different from our custom Append function above. Schematically, it's like this:

    func append(slice []T, elements ...T) []T

where T is a placeholder for any given type. You can't actually write a function in Go where the type T is determined by the caller. That's why append is built in: it needs support from the compiler.

What append does is append the elements to the end of the slice and return the result. The result needs to be returned because, as with our hand-written Append, the underlying array may change. This simple example

    x := []int{1,2,3}
    x = append(x, 4, 5, 6)
    fmt.Println(x)

prints [1 2 3 4 5 6]. So append works a little like Printf, collecting an arbitrary number of arguments.

But what if we wanted to do what our Append does and append a slice to a slice? Easy: use ... at the call site, just as we did in the call to Output above. This snippet produces identical output to the one above.

    x := []int{1,2,3}
    y := []int{4,5,6}
    x = append(x, y...)
    fmt.Println(x)

Without that ..., it wouldn't compile because the types would be wrong; y is not of type int.

##Initialization

Although it doesn't look superficially very different from initialization in C or C++, initialization in Go is more powerful. Complex structures can be built during initialization and the ordering issues among initialized objects, even among different packages, are handled correctly.

###Constants

 Constants in Go are just that—constant. They are created at compile time, even when defined as locals in functions, and can only be numbers, characters (runes), strings or booleans. Because of the compile-time restriction, the expressions that define them must be constant expressions, evaluatable by the compiler. For instance, 1<<3 is a constant expression, while math.Sin(math.Pi/4) is not because the function call to math.Sin needs to happen at run time.

In Go, enumerated constants are created using the iota enumerator. Since iota can be part of an expression and expressions can be implicitly repeated, it is easy to build intricate sets of values.

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

The ability to attach a method such as String to any user-defined type makes it possible for arbitrary values to format themselves automatically for printing. Although you'll see it most often applied to structs, this technique is also useful for scalar types such as floating-point types like ByteSize.

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

The expression YB prints as 1.00YB, while ByteSize(1e13) prints as 9.09TB.

The use here of Sprintf to implement ByteSize's String method is safe (avoids recurring indefinitely) not because of a conversion but because it calls Sprintf with %f, which is not a string format: Sprintf will only call the String method when it wants a string, and %f wants a floating-point value. 

###Variables

Variables can be initialized just like constants but the initializer can be a general expression computed at run time.

    var (
        home   = os.Getenv("HOME")
        user   = os.Getenv("USER")
        gopath = os.Getenv("GOPATH")
    )

###The init function

Finally, each source file can define its own niladic init function to set up whatever state is required. (Actually each file can have multiple init functions.) And finally means finally: init is called after all the variable declarations in the package have evaluated their initializers, and those are evaluated only after all the imported packages have been initialized.

Besides initializations that cannot be expressed as declarations, a common use of init functions is to verify or repair correctness of the program state before real execution begins.

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

##Methods
###Pointers vs. Values

As we saw with ByteSize, methods can be defined for any named type (except a pointer or an interface); the receiver does not have to be a struct.

In the discussion of slices above, we wrote an Append function. We can define it as a method on slices instead. To do this, we first declare a named type to which we can bind the method, and then make the receiver for the method a value of that type.

    type ByteSlice []byte

    func (slice ByteSlice) Append(data []byte) []byte {
        // Body exactly the same as above
    }

This still requires the method to return the updated slice. We can eliminate that clumsiness by redefining the method to take a pointer to a ByteSlice as its receiver, so the method can overwrite the caller's slice.

    func (p *ByteSlice) Append(data []byte) {
        slice := *p
        // Body as above, without the return.
        *p = slice
    }

In fact, we can do even better. If we modify our function so it looks like a standard Write method, like this,

    func (p *ByteSlice) Write(data []byte) (n int, err error) {
        slice := *p
        // Again as above.
        *p = slice
        return len(data), nil
    }

then the type *ByteSlice satisfies the standard interface io.Writer, which is handy. For instance, we can print into one.

    var b ByteSlice
    fmt.Fprintf(&b, "This hour has %d days\n", 7)

We pass the address of a ByteSlice because only *ByteSlice satisfies io.Writer. The rule about pointers vs. values for receivers is that value methods can be invoked on pointers and values, but pointer methods can only be invoked on pointers. This is because pointer methods can modify the receiver; invoking them on a copy of the value would cause those modifications to be discarded.

By the way, the idea of using Write on a slice of bytes is central to the implementation of bytes.Buffer.

##Interfaces and other types
###Interfaces

 Interfaces in Go provide a way to specify the behavior of an object: if something can do this, then it can be used here. We've seen a couple of simple examples already; custom printers can be implemented by a String method while Fprintf can generate output to anything with a Write method. Interfaces with only one or two methods are common in Go code, and are usually given a name derived from the method, such as io.Writer for something that implements Write.

A type can implement multiple interfaces. For instance, a collection can be sorted by the routines in package sort if it implements sort.Interface, which contains Len(), Less(i, j int) bool, and Swap(i, j int), and it could also have a custom formatter. In this contrived example Sequence satisfies both.

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

###Conversions

The String method of Sequence is recreating the work that Sprint already does for slices. We can share the effort if we convert the Sequence to a plain []int before calling Sprint.

    func (s Sequence) String() string {
        sort.Sort(s)
        return fmt.Sprint([]int(s))
    }

This method is another example of the conversion technique for calling Sprintf safely from a String method. Because the two types (Sequence and []int) are the same if we ignore the type name, it's legal to convert between them. The conversion doesn't create a new value, it just temporarily acts as though the existing value has a new type. (There are other legal conversions, such as from integer to floating point, that do create a new value.)

It's an idiom in Go programs to convert the type of an expression to access a different set of methods. As an example, we could use the existing type sort.IntSlice to reduce the entire example to this:

    type Sequence []int

    // Method for printing - sorts the elements before printing
    func (s Sequence) String() string {
        sort.IntSlice(s).Sort()
        return fmt.Sprint([]int(s))
    }

Now, instead of having Sequence implement multiple interfaces (sorting and printing), we're using the ability of a data item to be converted to multiple types (Sequence, sort.IntSlice and []int), each of which does some part of the job. That's more unusual in practice but can be effective.

###Interface conversions and type assertions

Type switches are a form of conversion: they take an interface and, for each case in the switch, in a sense convert it to the type of that case. Here's a simplified version of how the code under fmt.Printf turns a value into a string using a type switch. If it's already a string, we want the actual string value held by the interface, while if it has a String method we want the result of calling the method.

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

The first case finds a concrete value; the second converts the interface into another interface. It's perfectly fine to mix types this way.

What if there's only one type we care about? If we know the value holds a string and we just want to extract it? A one-case type switch would do, but so would a type assertion. A type assertion takes an interface value and extracts from it a value of the specified explicit type. The syntax borrows from the clause opening a type switch, but with an explicit type rather than the type keyword:

    value.(typeName)

and the result is a new value with the static type typeName. That type must either be the concrete type held by the interface, or a second interface type that the value can be converted to. To extract the string we know is in the value, we could write:

    str := value.(string)

But if it turns out that the value does not contain a string, the program will crash with a run-time error. To guard against that, use the "comma, ok" idiom to test, safely, whether the value is a string:

    str, ok := value.(string)
    if ok {
        fmt.Printf("string value is: %q\n", str)
    } else {
        fmt.Printf("value is not a string\n")
    }

If the type assertion fails, str will still exist and be of type string, but it will have the zero value, an empty string.

As an illustration of the capability, here's an if-else statement that's equivalent to the type switch that opened this section.

    if str, ok := value.(string); ok {
        return str
    } else if str, ok := value.(Stringer); ok {
        return str.String()
    }

###Generality

If a type exists only to implement an interface and has no exported methods beyond that interface, there is no need to export the type itself. Exporting just the interface makes it clear that it's the behavior that matters, not the implementation, and that other implementations with different properties can mirror the behavior of the original type. It also avoids the need to repeat the documentation on every instance of a common method.

In such cases, the constructor should return an interface value rather than the implementing type. As an example, in the hash libraries both crc32.NewIEEE and adler32.New return the interface type hash.Hash32. Substituting the CRC-32 algorithm for Adler-32 in a Go program requires only changing the constructor call; the rest of the code is unaffected by the change of algorithm.

A similar approach allows the streaming cipher algorithms in the various crypto packages to be separated from the block ciphers they chain together. The Block interface in the crypto/cipher package specifies the behavior of a block cipher, which provides encryption of a single block of data. Then, by analogy with the bufio package, cipher packages that implement this interface can be used to construct streaming ciphers, represented by the Stream interface, without knowing the details of the block encryption.

The crypto/cipher interfaces look like this:

    type Block interface {
        BlockSize() int
        Encrypt(src, dst []byte)
        Decrypt(src, dst []byte)
    }

    type Stream interface {
        XORKeyStream(dst, src []byte)
    }

Here's the definition of the counter mode (CTR) stream, which turns a block cipher into a streaming cipher; notice that the block cipher's details are abstracted away:

    // NewCTR returns a Stream that encrypts/decrypts using the given Block in
    // counter mode. The length of iv must be the same as the Block's block size.
    func NewCTR(block Block, iv []byte) Stream

NewCTR applies not just to one specific encryption algorithm and data source but to any implementation of the Block interface and any Stream. Because they return interface values, replacing CTR encryption with other encryption modes is a localized change. The constructor calls must be edited, but because the surrounding code must treat the result only as a Stream, it won't notice the difference.

###Interfaces and methods

Since almost anything can have methods attached, almost anything can satisfy an interface. One illustrative example is in the http package, which defines the Handler interface. Any object that implements Handler can serve HTTP requests.

    type Handler interface {
        ServeHTTP(ResponseWriter, *Request)
    }

ResponseWriter is itself an interface that provides access to the methods needed to return the response to the client. Those methods include the standard Write method, so an http.ResponseWriter can be used wherever an io.Writer can be used. Request is a struct containing a parsed representation of the request from the client.

For brevity, let's ignore POSTs and assume HTTP requests are always GETs; that simplification does not affect the way the handlers are set up. Here's a trivial but complete implementation of a handler to count the number of times the page is visited.

    // Simple counter server.
    type Counter struct {
        n int
    }

    func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
        ctr.n++
        fmt.Fprintf(w, "counter = %d\n", ctr.n)
    }

(Keeping with our theme, note how Fprintf can print to an http.ResponseWriter.) For reference, here's how to attach such a server to a node on the URL tree.

    import "net/http"
    ...
    ctr := new(Counter)
    http.Handle("/counter", ctr)

But why make Counter a struct? An integer is all that's needed. (The receiver needs to be a pointer so the increment is visible to the caller.)

    // Simpler counter server.
    type Counter int

    func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
        *ctr++
        fmt.Fprintf(w, "counter = %d\n", *ctr)
    }

What if your program has some internal state that needs to be notified that a page has been visited? Tie a channel to the web page.

    // A channel that sends a notification on each visit.
    // (Probably want the channel to be buffered.)
    type Chan chan *http.Request

    func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
        ch <- req
        fmt.Fprint(w, "notification sent")
    }

Finally, let's say we wanted to present on /args the arguments used when invoking the server binary. It's easy to write a function to print the arguments.

    func ArgServer() {
        fmt.Println(os.Args)
    }

How do we turn that into an HTTP server? We could make ArgServer a method of some type whose value we ignore, but there's a cleaner way. Since we can define a method for any type except pointers and interfaces, we can write a method for a function. The http package contains this code:

    // The HandlerFunc type is an adapter to allow the use of
    // ordinary functions as HTTP handlers.  If f is a function
    // with the appropriate signature, HandlerFunc(f) is a
    // Handler object that calls f.
    type HandlerFunc func(ResponseWriter, *Request)

    // ServeHTTP calls f(c, req).
    func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
        f(w, req)
    }

HandlerFunc is a type with a method, ServeHTTP, so values of that type can serve HTTP requests. Look at the implementation of the method: the receiver is a function, f, and the method calls f. That may seem odd but it's not that different from, say, the receiver being a channel and the method sending on the channel.

To make ArgServer into an HTTP server, we first modify it to have the right signature.

    // Argument server.
    func ArgServer(w http.ResponseWriter, req *http.Request) {
        fmt.Fprintln(w, os.Args)
    }

ArgServer now has same signature as HandlerFunc, so it can be converted to that type to access its methods, just as we converted Sequence to IntSlice to access IntSlice.Sort. The code to set it up is concise:

    http.Handle("/args", http.HandlerFunc(ArgServer))

When someone visits the page /args, the handler installed at that page has value ArgServer and type HandlerFunc. The HTTP server will invoke the method ServeHTTP of that type, with ArgServer as the receiver, which will in turn call ArgServer (via the invocation f(c, req) inside HandlerFunc.ServeHTTP). The arguments will then be displayed.

In this section we have made an HTTP server from a struct, an integer, a channel, and a function, all because interfaces are just sets of methods, which can be defined for (almost) any type.

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


Build version go1.1.2.
Except as noted, the content of this page is licensed under the Creative Commons Attribution 3.0 License, and code is licensed under a BSD license.
