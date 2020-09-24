# Introduction to importing modules in D

## Introduction

Working with modules is a basic and fundamental part of programming, and provides the programmer with an valuable source of predefined functionality. The `import` directive in D provides an easy way of accessing these resources, and this article outlines the various ways of using it.

## Give me everything

This approach imports everything in the `std.stdio` module for use in the current scope:

```d
import std.stdio;
```

### Scope sensitivity

Be aware that imports are scope and order sensitive, meaning that if you import a module or part of a module into a scope it is only available for use in that scope from the point that you gave the import command, it is not automatically available for use globally or even within the scope before it was imported. This means that this code will give a definition error:

```d
void main()
{
  writeln("Hello world!");
  import std.stdio;
}
```
It occurs because import of Phobos's I/O library occurs *after* calling the `writeln` function. The code below will also give a definition error because the import is only for the scope of `hello`:

```d
auto hello()
{
  import std.stdio;
  writeln("Hello World!");
}

void main()
{
  hello();
  writeln("Hello World!");
}
```

## Importing specific items from a module

If we only need specific functions from a module for instance `write` and `writeln` we can use the pattern:

```d
import std.stdio: write, writeln;
```

If we want to import items from a module using another name, for instance, importing all the floating point implementations of `sin` functions, from  C math in the `core.stdc.math` module under `sin` we can use the pattern:

```d
import core.stdc.math: sin = sinf, sin, sin = sinl;
```

In the above example, the various functions overload `sin` with different signatures for each floating point type.

## Named module imports and pasting code from a file

D also allows you to paste code from a file as string that is interpreted to legal code during compile time using a **string mixin**. This is different from importing code in the previous examples, because the file from which we import code is not a module but just an informal script containing D code.

Consider the code below in the file `hello.d`:

```d
void hello()
{
  io.writeln("Hello from imported function.");
}
```

`hello.d` calls `io.writeln` not defined in the file. Attempting to compile the code on it's own will not work. But we can include it in a script that defines `io.writeln`:

```d
import io = std.stdio;
mixin(import("hello.d"));

/*
  To compile:
  dmd import.d -J="." && ./import
*/

void main()
{
  hello();
}
```

You can see that we assigned the whole `std.stdio` module to `io`, and used that as a prefix in our code to access its `writeln` function.

*Note that we must specify the folder where the script is located using the `-J` flag in our compilation pattern `dmd import.d -J="."` (in the DMD compiler).* 

## Static import

Static import means that we must use the full module to item name when addressing any part of the module:

```d
static import std.stdio;


void main()
{
  std.stdio.writeln("Hello World!");
}
```

## Public import

By default, the `import` keyword imports modules privately, meaning that the imported functions are only available to the module immediately importing the functions. Using `public import` will import the module to the current scope but will also import that module or specified subset to all the scripts that reference the importing module. In other words public import is as if the imported module or items where written in the scope that it is imported in. For example in `script.d`:

```d
public import std.stdio: writeln;
```

This means that the function `writeln` will be available in `script.d` but also in **all** scripts, modules, and libraries that import `script.d`.

## Items marked `"private"` are not imported

Items marked with `private` will not be imported. For example the script below is in a file called `bye.d`

```d
module bye; // This is how you declare a module name in D

private void goodbye()
{
  import std.stdio: writeln;
  writeln("You can not use this function in another library.");
}
```

We import it in `script.d`:

```d
import bye;

/*
  To compile:
  dmd script.d bye.d && ./script
*/

void main()
{
  goodbye(); //will not compile!
}
```

`script.d` will not compile because the function `goodbye` is not exported and so is not available for use.

By default D will create a module for script with the same name as the script file name, a script with no `module`  declaration will be given a module name the same as the file name so a script named `bye.d` will be named as the module `bye` by default.

That's it!
