# Introduction to importing techniques in D

This brief article describes various ways of importing functions from libraries and scripts in D.

Firstly there is the very simple approach that imports all the functions in a module:

```
import std.stdio;
```
The above will import all the exportable items in `std.stdio` into the current script. If we only need two functions for instance `write` and `writeln` we can use the pattern:

```
import std.stdio: write, writeln;
```

If we want to import items/functions using another alias we can use the pattern:

```
import core.stdc.math: sin = sinf, sin, sin = sinl;
```
In the above example, we overload the `sin` alias with different functions for each floating point type.

D also allows you to import code from a file as string that is parsed to legal code during compile time using it's `mixin`s. Consider the code below:

```
//hello.d
void hello()
{
  io.writeln("Hello from imported function.");
}
```

`hello.d` calls `io.writeln` not defined in the file. Attempting to compile the code on it's own will not work. But we can include it in a script that defines `io.writeln` using a mixin:

```
static import io = std.stdio;
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

Note here that we must specify the folder where the script is located using the `-J` flag in our compilation pattern `dmd import.d -J="."`. We also have a new import pattern where we can denote a whole library as another alias `io`, `static import` imports the library but we can only access the items in the library using the pattern `io`. To use the default library name we import using `static import std.stdio;` then we access library items for example `writeln` using `std.stdio.writeln`.

By default the `import` keyword imports modules privately, meaning that the imported functions are only available to the module immediately importing the functions. Using `public import` will import the library/module to the importing module but will also make that library/module available to all the scripts that reference the importing module. Wow, that's a mouthful. By way of example consider the following:

```
//myscript.d
public import std.stdio: writeln;
```

This means that the function writeln will be available in `myscript.d` but also in **all** scripts, modules, and libraries that import `myscript.d`.

To stop a function in a module from being imported, mark it with private. For example:

```
// bye.d
module bye; // This is how you declare a module name in D

private void goodbye()
{
  import std.stdio: writeln;
  writeln("You can not use this function in another library.");
}
```

```
//import.d
import bye;

/*
  To compile:
  dmd import.d bye.d && ./import
*/

void main()
{
  //goodbye(); //will not compile
}
```

Also by default D will create a module for script with the same name as the script file name, a script with no `module`  declaration will be given a module name the same as the file name so a script named `bye.d` will be named as the module `bye` by default.

That's it.
