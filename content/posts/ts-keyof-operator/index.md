+++
title = "`Keyof` unlocks door to flexible TS code."
publishDate = 2024-07-10T00:00:00+05:45
lastmod = 2024-07-12T10:24:17+08:45
tags = ["programming", "typescript"]
categories = ["languages", "programming"]
draft = true
featuredImage = "images/keyof-thumbnail.jpg"
featuredImagePreview = "./posts/ts-keyof-operator/images/keyof-thumbnail.jpg"
+++


As someone who has been involved in software development for over 3 years, I have primarily worked with JavaScript -*and nowadays Typescript mostly*- although I have also used languages like Python for specific projects. In this article, I want to share my initial impressions of the Go programming language as a web developer. I don't intend for this to be a negative critique of JavaScript, as there are already plenty of those. Instead, I will highlight a few aspects of Go that I find particularly interesting and cool.

Simple type system 🎹
------------------

I really love TypeScript. It has completely changed the way I write Frontend by reducing the need for excessive console logging. However, there are times when I find the type system to be quite complex. Especially when I'm working on large projects, I often have to deal with convoluted type manipulations. And to make matters worse, the error messages can be really confusing and not very helpful. It can be quite frustrating trying to understand what went wrong.

{{< figure src="images/really-long-ts-error.png" caption="*Very long Typescript Error* 😭" >}}

Go's type system is simple and straightforward. It includes common primitives like int and string, as well as arrays, slices, structs, interfaces, and functions. Generics were also [introduced later on](https://go.dev/blog/intro-generics). The compiler is highly efficient and provides concise and helpful error messages in most cases.

A single way to do things 🚀
-------------------------

While it may initially seem limiting, the fact that Go provides only one approach for each task makes it a straightforward language to work with. This simplicity is especially beneficial for large teams working on extensive codebases. Whether a junior or senior developer, they would produce the same code when solving the same problem.

In Go, there is only one way to create a loop:

{{<highlight go>}}
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
    fmt.Println(sum)
{{</highlight>}}
The `init` and `post` statements are not required in Go. By removing the semicolons, the `for` loop can be transformed into a `while` loop:

{{<highlight go>}}
sum := 1
for sum < 1000 {
    sum += sum
}
    fmt.Println(sum)
    {{</highlight>}}


If you omit the loop condition, you get an inifinite loop:

    {{<highlight go>}}
    for {
    	// runs forever
    }
    {{</highlight>}}

Explicit error handling 🚨
--------------------

In JavaScript development, it is often the case that we return errors as regular JavaScript objects instead of throwing instances of the built-in `Error` class. The act of throwing should only be used when intentionally crashing the program. This is because JavaScript is [single-threaded and synchronous](https://groovetechnology.com/blog/why-javascript-is-single-threaded/), so throwing an error will stop the execution of the program. This can be problematic in some cases, especially when working with asynchronous code. Additionally, JavaScript has a built-in `try...catch` statement that can be used to catch errors and handle them gracefully.

{{<highlight javascript>}}
    try {
      throw new Error('This is an error');
    } catch (error) {
      console.error(error.message);
    }
{{</highlight>}}

In Go, [error handling](https://go.dev/blog/error-handling-and-go) is built-in and follows a different approach. Errors are returned as values, and they are explicitly handled using `if err != nil` checks. If a program needs to crash, the `panic` keyword can be used. Additionally, Go provides the `recover` statement to recover from a panic. Although writing error handling code in this manner may seem complex, it greatly enhances readability, which is a valuable trade-off in my opinion.

The `defer` statment: A simple way to handle cleanup tasks 🧹
-----
For those of you familiar to React's useEffect's cleanup function, the `defer` statement in Go is similar. The defer statement in Go allows you to postpone the execution of a function until the surrounding function finishes. Let's take a look at an example where I open a file to copy its contents to another file:

{{<highlight go "hl_inline=true, hl_Lines=6-9">}}
    func CopyFile(dstName, srcName string) (written int64, err error) {
        src, err := os.Open(srcName)
        if err != nil {
            return
        }
        dst, err := os.Create(dstName)
        if err != nil {
            return
        }
        written, err = io.Copy(dst, src)
        dst.Close()
        src.Close()
        return
    }
{{</highlight>}}

{{< admonition type=danger title="This works, but there's a bug" open=true >}}
If the call to `os.Create` fails, the function will return without closing the src file. This can be easily fixed by adding a call to `src.Close` before the second return statement, but if the function were more complex the problem might not be so easily noticed and resolved. By introducing defer statements we can ensure that the files are always closed:
{{< /admonition >}}

{{<highlight go "hl_Lines=6" >}}
    func CopyFile(dstName, srcName string) (written int64, err error) {
        src, err := os.Open(srcName)
        if err != nil {
            return
        }
        defer src.Close()

        dst, err := os.Create(dstName)
        if err != nil {
            return
        }
        defer dst.Close()

        return io.Copy(dst, src)
    }
{{</highlight>}}


Defer statements allow us to think about closing each file right after opening it, guaranteeing that, regardless of the number of return statements in the function, the files will be closed.

This example is inspired from an excellent article: [defer, panic, and recover](https://go.dev/blog/defer-panic-and-recover) by Andrew Gerrand. It’s a fantastic read and it expands on the rules of `defer` and how it works in tandem with other unique control flow statements like `panic` and `recover`.

Type conversion is explicit 🔄
---------------------------
One of the main subjects of memes in JavaScript is its ridiculous type coercion. JavaScript attempts to automatically convert the types of values based on the operation being performed. Although type coercion can be useful in certain situations, it frequently results in bugs and confusion for beginners.

{{< figure src="images/5d35082832a25ca9c3f971f42c98bcde8c46cca2.png" >}}

**In Go**, explicit type conversion is required when assigning values between different types. Attempting to assign a `string` to an `int` or perform operations between mismatched types will result in a compilation error. Although this may add some complexity to writing Go code, it greatly enhances code readability and maintainability, which is a common characteristic of the language.


Give `break` a break 🚫
---------------------------------------------------------

Here’s a JavaScript switch statement from [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch#try_it):

{{<highlight javascript>}}
    const expr = 'Papayas';
    switch (expr) {
      case 'Oranges':
        console.log('Oranges are $0.59 a pound.');
        break;
      case 'Mangoes':
      case 'Papayas':
        console.log('Mangoes and papayas are $2.79 a pound.');
        break;
      default:
        console.log(`Sorry, we are out of ${expr}.`);
    }
{{</highlight>}}

Now in Go:
{{<highlight go>}}
    expr := "Papayas"
    switch expr {
    case "Oranges":
    	fmt.Println("Oranges are $0.59 a pound.")
    case "Mangoes", "Papayas":
    	fmt.Println("Mangoes and papayas are $2.79 a pound.")
    default:
    	fmt.Printf("Sorry, we are out of %s.", expr)
    }
{{</highlight>}}

In Go, the switch statement works differently compared to C, Java, JavaScript, and PHP. In Go, only the selected case is executed, unlike other languages where all subsequent cases are also executed. This eliminates the need for a break statement at the end of each case, as required in those languages, as Go handles it automatically.

Pointers 🧭
--------

In Go, function arguments are passed by value, which means they are copied by default. This rule also applies to arrays and structs. However, you have the option to explicitly define a function that takes a pointer as an argument. Pointers store the memory address of a value. Here's an example taken from the [official Go tutorial](https://go.dev/tour/moretypes/1):

{{<highlight go>}}
    package main
    import "fmt"
    func main() {
    	i, j := 42, 2701
    	p := &i         // point to i
    	fmt.Println(*p) // read i through the pointer
    	*p = 21         // set i through the pointer
    	fmt.Println(i)  // see the new value of i

    	p = &j         // point to j
    	*p = *p / 37   // divide j through the pointer
    	fmt.Println(j) // see the new value of j
    }
{{</highlight>}}

Again, this makes Golang so easy to read and maintain.



Still a Long Way to Go 🚶‍♂️
----------------

I've just begun my journey with Go. If I had to put it in words, I'd say Go is easy. Really, really easy. When you look at a Go program, you can almost always tell what it's doing. Recently, I have had the pleasure of writing Golang at my work for a RFID communication server. I was tasked to convert the pre-existing Windows x64 binary into a background daemon. Thanks to Go's rich ecosystem of libraries, I was able to use this [`service`](https://github.com/kardianos/service) package to compile the RFID websocket server project into a Windows service. Go works so fast. It's hard to wrap my head around the fact that Go, even though it cleans up its own memory, can run this fast. It's been the backbone for building important software like [Docker](https://www.slideshare.net/slideshow/docker-and-go-why-did-we-decide-to-write-docker-in-go/28015076), the [Cockroach database](https://github.com/cockroachdb/cockroach) and many more.

This is just the start for me. I'll share more about Go as I learn more about it. I can't wait to see what I can make with this super easy language. Wish me luck! ✌

