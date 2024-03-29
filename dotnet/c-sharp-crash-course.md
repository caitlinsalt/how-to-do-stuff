# C#: A Crash Course

This extended essay, extended presentation, or whatever it becomes, is a brief guide to C# for developers who are already familiar with coding, but need to learn the basics of C# (and .NET in general) quickly.  It makes no claims to completeness, but should hopefully introduce you to basic principles and idioms in a rather information-dense fashion, whilst at the same time being a little bit less terse than a cheat sheet.  I've always found there is little benefit in memorising a cheat sheet of magic phrases unless you know the principles underlying them, so I've tried to strike a balance between terseness and explanation.

## Basic principles - the CLI

### A high level overview

C# is a compiled-interpreted language that was created as part of the .NET Framework, as a language to run on top of the Common Language Infrastructure (CLI) standard.  It is normally compiled to CIL (Common Intermediate Language) bytecode, which is an instruction set that targets a virtual machine called the VES (Virtual Execution System).  The bytecode was designed to be both high-level (for example, it has a `new` instruction which combines both heap allocation and constructor calling) and amenable to JIT compilation.

The various nested parts of the CLI standard can be slightly confusing.  The CLI standard itself includes all of the following:

- The Common Type System (CTS).
- The Common Language Specification (CLS).
- The Common Intermediate Language (CIL, originally Microsoft Intermediate Language or MSIL).
- The Virtual Execution System.

An implementation of the CLI standard and the necessary libraries is called a Common Language Runtime (CLR).  Most implementations also fulfil a version of an extended standard, ".NET Standard", which is the CLI standard plus certain additional libraries.  However, even the latest version of the .NET Standard falls short of the library features found in the most recent Microsoft implementations (.NET 5 and .NET 6), and the .NET Standard is not going to be updated to take these features into account, so it is now something of a dead letter.

Aside fromo these standards, which concern the underlying runtime, the C# language also has its own standards documents and version numbers, and although a given version of C# is often tied to a particular .NET implementation, the numbers don't match (for example C# 10 is the language version that matches .NET 6).  I've tried to explain how the various versions and standards marry up in a table below.

C# is at its heart a strongly-typed language.  The CTS describes the type system of the CLI; and is effectively the core of the CLI because it determines a huge amount of basic fundamental behaviour.  In the CTS and therefore the CLI, all code must belong to a type.  The CLS is a subset of the CTS, which ensures compatibility between CIL code written in different source languages.  C# is a CLS-conforming language, and can consume libraries written in other CLS-conforming languages; but it is also possible to write non-CLS-conforming code in C# or in other languages.  If you write a library in C# using non-CLS-conforming language features, code written in other languages might not be able to call it, and vice-versa.

When creating code in Visual Studio you must choose a "target framework" - this is the particular CLI implementation (and version) that you expect to run your code.  The .NET Standard target outputs code which should be compatible with any implementation that meets the given version of that standard; other targets will produce code that will only run on the specific implementation (or sometimes, a newer version of the same implementation).  Although all of the implementations meet the CLI specification, there are many things that it does not cover or that have in some way become out of date.  These can include (but are not limited to):

- how programs are started
- configuration file formats
- details of the build process
- non-standard libraries provided with the framework.
- package management

In particular, GUI libraries and other similar-level frameworks are not standardised, so their ability will depend on the implementation you are using.

Over the past twenty years Microsoft have produced two major CLR implementations: the .NET Framework, and .NET, which was formerly called .NET Core.  The former was a Windows-based closed-source implementation; the latter is cross-platform and largely open-source.  The .NET Framework is still under long-term support but will not receive new feature releases.  For clarity, when I say ".NET" I'm referring to the current CLR implementation that was formerly called ".NET Core", and not to CLR implementations in general.

I mentioned above that the C# language and the .NET implementation have different version numbers.  In fact, all of the following have different numbering sequences, some of them relatively close together and some of them radically different:

- the CLI standard
- the C# language
- the .NET Framework CLI implementation
- the .NET (or .NET Core) CLI implementation
- the .NET Standard API standard

In general, below, most version number references in this document are to the relevant version of the C# language, and I've tried to make it clear what any version number I mention refers to.  However, here's an approximate correspondance table.  An exact one would be difficult to express succinctly.

| CLI version | C# version    | .NET Framework version | .NET (or .NET Core) version | .NET Standard version |
| ----------- | ------------- | ---------------------- | --------------------------- | --------------------- |
| 1.0         | 1.0           | 1.0                    | N/A                         | N/A                   |
| 1.1         | 1.1           | 1.1                    | N/A                         | N/A                   |
| 2.0         | 2.0           | 2.0                    | N/A                         | N/A                   |
|             |               | 3.0                    | N/A                         | N/A                   |
|             | 3.0           | 3.5                    | N/A                         | N/A                   |
| 4           | 4.0           | 4.0                    | N/A                         | N/A                   |
|             | 5.0           | 4.5                    | N/A                         | 1.0                   |
|             |               |                        | N/A                         | 1.1                   |
|             |               | 4.5.1                  | N/A                         | 1.2                   |
|             | 6.0           | 4.6                    | N/A                         | 1.3                   |
|             |               | 4.6.1                  | 1.0                         | 1.4                   |
|             | 7.0           | 4.6.2                  | 1.1                         |                       |
|             | 7.1           | 4.7                    | 2.0                         |                       |
|             | 7.2           | 4.7.1                  |                             |                       |
|             | 7.3           | 4.7.2                  | 2.1                         | 2.0                   |
|             |               |                        | 2.2                         |                       |
|             |               | 4.8                    |                             |                       |
|             | 8.0           |                        |                             |                       |
|             |               | N/A                    | 3.0                         | 2.1                   |
|             | 9.0           | N/A                    | 5                           |                       |
|             | 10.0          | N/A                    | 6                           |                       |

I've omitted a couple versions of .NET Standard between 1.4 and 2.0 which were implemented by .NET Core from version 1, but not implemented by .NET Framework until version 4.7.2.  .NET Framework 4.8 supports some features of .NET Standard 2.1 but does not support all of them.  Microsoft have announced that future versions of .NET will continue supporting .NET Standard 2.1, but further versions of .NET Standard will not be produced.

### Building and running code

Most C# developers use Visual Studio.  In that program, the top level item&mdash;the file that you open to start work&mdash;is the solution, stored in a `.sln` file.  it contains projects, which can be of various kinds: C# projects, other .NET projects, or other kinds of project entirely.  When you create a new project there are a large number of different C#-related templates to set up projects with different purposes, which set the target framework and various different configuration options, and usually create some amount of boilerplate code, but all of them store the project's details in a `.csproj` file.  Other languages and kinds of project use different file extensions for their project files.  The project file is an XML file that defines the project's target framework, various project-level settings, any non-standard variations on the build process, and potentially a large amount of other information.  In .NET Framework the project file also referenced all of the source code files belonging to the project; in .NET, the project file only contains references to source files that have some sort of non-default setting or behaviour, with .NET assuming that all files inside the project folder (other than those in build output folders) belong to the project and have default behaviour.  In general, partly become of this, .NET `.csproj` files are much simpler and shorter than .NET Framework ones.  

You can modify the build process by hand-editing the `.csproj` file, but this is a relatively rarely-required task.  It can however be useful to alter project-level settings which are not surfaced in the IDE.

Visual Studio handles building code itself, but Microsoft also supply command-line build tools which will also parses your `.sln` and `.csproj` files and carries out the required build steps.  For the .NET Framework this was `MSBuild.exe`; for .NET this is done using the `dotnet` utility with the `dotnet msbuild` command.  The latter has preserved command-line-argument compatibility with the former.  There is also a `dotnet build` command which does not preserve compatibility but is simpler to use.  If you set up any sort of CI and/or CD pipeline it will probably use `dotnet` (or MSBuild) to control and run the build process, rather than Visual Studio.  As an aside, [I've noticed in the past that](http://www.symbolicforest.com/blog/2020/11/13/technical-advice-post-of-the-week/) the .NET Framework MSBuild parser for `.sln` files is more restrictive than the Visual Studio parser and will throw errors on some `.sln` syntax errors than Visual Studio will silently ignore; this can lead to issues where CI builds are broken but developers cannot reproduce the error.  The solution to this is to make a zero-effect change to the solution, to convince Visual Studio that the `.sln` file needs to be written to disk; it will silently correct the syntax error(s) as it does so.

Although as I said most C# developers use Visual Studio, .NET has made it much simpler to develop in C# using other tools, as the `dotnet` command includes tools for carrying out a lot of tasks which were previously hard to do manually: setting up new projects, for example, or building and publishing code.  If you don't want to learn the whole of Visual Studio, it is entirely possible now to use a more Unix-like editor-and-commandline workflow, such as Visual Studio Code and Powershell.  JetBrains have also introduced a paid-for .NET IDE called "Rider".  I wouldn't recommend trying to use .NET Framework without Visual Studio.

Both .NET and .NET Framework ship the language build tools as part of the framework, so all computers that are capable of running .NET (or .NET Framework) applications are, in theory, capable of building them too.  This includes MSBuild functionality, and the C# compiler itself (`csc.exe` on .NET Framework, `csc.dll` on .NET).  As above, this is relatively straightforward to do with .NET but is rather more difficult with the .NET Framework, generally requiring a lot of wrestling with command-line options.

On .NET Framework, code with an entry point compiles to an `.exe` file, and code without an entry point compiles to `.dll` files.  With .NET, all code normally compiles to `.dll` files, and `.dll` files whose code has an entry point can be run using the `dotnet` tool. The project templates for things like command-line programs will generally also configure the build to output directly-executable files.  A .NET `.exe` file will either include a copy of the VES code within the executable, or code to find and run an installed copy of the VES.

## Basic principles - syntax, types, assemblies and classes

### Basic syntax

#### Similarities

C# is a C-derived language, so the following general rules similar to C, C++ and Java apply:

- Single statements are terminated with `;`.
- Block statements are delimited with `{` and `}`.
- In general the language is white space agnostic, but there are standards.
- The `=` operator is used for assignment, and the `==` operator for comparison.
- String constants are delimited by `"`, character constants by `'`, hex constants are prefixed `0x` and octal constants prefixed `0`.
- In most cases operators have approximately the same meanings as in the other languages, including the ternery conditional operator.
- Variables are declared before or when they are initialised.
- In most cases, C# has similar scoping rules to the other languages mentioned.
- The flow-control keywords `if`, `else`, `for`, `do` and `while` work in a similar way to the other languages.  There is also a `foreach` loop.
- Members are accessed using the `.` operator.
- Indexing is normally zero-based.

Although C# uses `*` for pointer dereferencing like C and C++, pointers are only encountered very rarely in C# code and can only be used under relatively strict conditions.

Identifiers are case-sensitive, can contain alphanumeric characters or underscores, and cannot start with a decimal digit.

One of the requirements of the CLS is that languages must provide an escaping system so that a keyword or other reserved word can be used as an identifier; this means that you don't have to worry about what words are reserved in other languages that might want to consume your code.  In C# this is done with the `@` character.  In code, for example, `if` is a keyword, but `@if` is the identifier whose actual name in the compiled metadata will be "if".  In general it is best to avoid identifiers that require `@`, but you might do (and might need to) for compatibility with other systems.

Note that the set of valid CLS (and C#) identifiers is, even without considering `@`, a subset of all valid CLI identifiers.  This enables the compiler to generate and emit code which is not accessible by name from within user-written code, because its identifier is valid within the CLI but not within the language.  The compiler uses this, for example, to name anonymous methods and types, among other things.  You can only see these generated identifiers by decompiling the CIL code.

C# doesn't have a preprocessor like C and C++ do&mdash;but despite this, it *does* have preprocessor directives!  Thehy are similar in nature to those in C and C++ and they also start with `#`.  Their primary uses are:
- Conditional compilation (with `#if` and its associated friends).
- Separating a code file into sections that a supporting editor can collapse or expand for display (with `#region` and `#endregion`).
- Turning types of compiler warning on and off (with `#pragma warning`). 

Since C# 7.0 the identifier `_` has had a special meaning as the "discard variable name" when it has not been declared as anything else.  Although this feature was introduced to the language in such a way that it did not break any existing code which used `_` as a normal identifier, it is advisable not to use it as a normal identifier in new code in order to avoid confusion.  Discard variables are described further below.

#### Comments

As in Java and C++, C# supports both `/* ... */` block comments and `//` single-line comments.  The latter are more commonly used.

There are also `///` and `/** ... */` comments which are similar in concept to `/** ... */` doc-comments in Java.  These comments appear before definitions and contain XML in a particular schema.  The compiler can extract the content of these comments at compile-time and write them out to an XML file, which then can be processed by other tools to generate documentation files.  Even if you don't use them to generate separate documentation, Visual Studio will read the comments and display the documentation in tooltips when you mouse over calling code.  Here's an example of what it looks like:

```
/// <summary>
/// A biscuit storage class
/// </summary>
class BiscuitStore
{
    /// <summary>
    /// Constructor with factory parameter
    /// </summary>
    /// <param name="factory">Biscuit factory implementation - see <see cref="IBiscuitFactory"/>.</param>
    BiscuitStore(IBiscuitFactory factory)
    {
        // ...
    }

    // ...
}
```

In real-life code these comments usually use `///`, with `/** ... */` being comparatively rare.  If you did come across it, it would look like this:

```
/**
 * <summary>
 * A biscuit storage class
 * </summary>
 **/
class BiscuitStore
```

The initial `*` on each line is removed, to allow for this style of formatting, but they all must be vertically in line, with the same indent, for this to work properly.

#### Garbage collection

The CLR is a garbage-collected environment, so C# is a garbage-collected language, at least for the vast majority of data created by most programs.  Data that is under the wing of the garbage collector is sometimes referred to as "managed data" or "managed objects".  Generally speaking all objects created from within C# code itself are managed objects, but objects created from external libraries may be "unmanaged objects" and may have to be manually closed or disposed: for example, open file handles or database connections.  Managed objects which look after unmanaged objects are called "disposable objects" and should contain a certain amount of largely-boilerplate code (which Visual Studio can insert an outline template for) to ensure the garbage collector handles the unmanaged references properly.  Managed objects which look after disposable objects should also be disposable objects themselves.

The term "unmanaged type" is also used in some contexts to refer to data created from within C# code that does not contain or refer to objects, because again the garbage collector does not need to get involved in cleaning these up.

I will explain about garbage collection and the disposable type pattern in more detail later, after we've talked more about the type system.  For now, all you need to know is that in general you don't need to worry *too* much about manually cleaning up memory.

### Types and assemblies

#### Types and namespaces

All code in C# (in fact, in all CLI languages) belongs to a type, and every expression and every declared storage location also has a type.  Each type belongs to a namespace, and has a fully-qualified name which consists of the namespace name, a dot, and the type name.  For example, the `System.IO` namespace contains the `Path` class, whose fully-qualified name is `System.IO.Path`.  Namespaces are lexically hierarchical, with name segments separated by dots, and the namespace hierarchy within a project by convention, normally reflects the folder hierarchy of the source files in that project.  This isn't strictly compulsary, but is recommended.

Traditionally, each source-code file contains a top level `namespace` block, which contains type definitions within it, as follows:

```
namespace MyApplication
{
    class Program
    {
        // ...
    }
}
```

The above fragment defines the class `MyApplication.Program`.  Some of the examples below in this document include the `namespace { ... }` block for completeness; others omit it for conciseness.   From C# 10 onwards, the namespace block is not needed if you only have one namespace in a file, which&mdash;given the convention that the namespace hierarchy matches the project source code folder hierarchy&mdash;is nearly always the case.

```
namespace MyApplication;
class Program
{
    // ...
}
```

Within a given namespace and its hierarchical descendents, types in that namespace can be referred to by just the name of the type, but types in unrelated namespaces, by default, have to be referred to by their fully-qualified name.  The `using` directive, within a code file, specifies namespaces to be lexically imported, so that their types do not need to be specified by their fully-qualified names.  Visual Studio normally adds a number of `using` directive to the top of each new code file by default, such as `using System;` and `using System.Linq;`  The `using` directive can also be used to alias a type or a namespace, with the syntax `using Alias = Some.Actual.Type;`, which can be useful to avoid naming clashes.

From C# 10 onwards, the `global using` directive is a `using` directive which occurs only in one code file but applies (in most cases) to all of the code in the same project.  The .NET 6 templates which use `global using` generally follow the pattern of including a file called `Usings.cs` which contains `global using` directives and nothing else.  Global `using` directives have to be the first code in any file they are in.

Note: don't confuse `using` directives with `using` statements, which also use the same keyword.  `using` statements are covered below when we talk about the disposable pattern.

If namespaces are related&mdash;that is, have the same root&mdash;then partially-qualified type names can be used, by removing common segments at the start of the namespace name.  For example, if a project has the namespaces `MyApplication` and `MyApplication.Helpers`, then within the class `MyApplication.Thing` the class whose fully-qualified name is `MyApplication.Helpers.ThingHelper` can be referred to with the partially-qualified name `Helpers.ThingHelper`, and in the other direction, within the `MyApplication.Helpers.ThingHelper` class the `MyApplication.Thing` class can be referred to as just `Thing`.

Types can be split into "value types" and "reference types", depending on which parameter-passing convention applies to them.  Classes, interfaces and delegates are reference types, structs and enums are both value types, and record types (which were only introduced relatively recently) can be either.  The built-in numeric types (and the `bool` type) are kinds of struct; the built in `string` type is a reference type and behaves like a class.  Classes and interfaces have a defined inheritance hierarchy: a class can inherit any single non-`sealed` class and can implement any number of interfaces, a reference-type record (or "record class") can inherit from any other single reference-type record, and an interface can inherit any number of other interfaces; this is described more fully below.  Other kinds of type have a fixed inheritance hierarchy which can't be changed by the developer.  All types ultimately inherit from the `object` type, which means that even on, say, a numeric constant, you can call `object` methods such as `.ToString()` and `.Equals()`.

Value record types are referred to as "record structs"  Reference record types are sometimes referred to as "record classes".  However, record classes are not classes, and record structs are not structs.  Because record classes were introduced in C# 9 and were just called "records" at that point, if somebody talks about "records", they may be referring to records in general, or to record classes specifically, depending on context.  Record structs were introduced in C# 10.

Traditionally in C#, one fundamental difference between reference and value types is that reference type expressions and variables can have the value `null` and value type expressions and variables cannot.  In the CLR this is still the case.  However, since C# 2, the language has supported declaring value types as nullable by suffixing the type with a `?` when declaring it: for example, the nullable version of `int` becomes `int?`.  Under the hood this is done by wrapping the value in another struct whose type is (in this example) `Nullable<int>`, and `int?` is shorthand for that type.  For any nullable value type, the `Value` property gives the underlying value cast to a non-nullable type, and the `HasValue` property is `false` if the value is `null` and `true` if not.

Reference types in the CLR are always nullable, but C# 8 introduced the concept of a "nullable context", a section of code where the compiler treats reference types as non-nullable by default unless they are flagged as nullable with the `?` suffix, and can raise an error or a warning (according to the kind of context) if a `null` value is potentially assigned to a non-nullable reference.  Initially this feature had to be deliberately chosen by the developer.  From .NET 6 onwards, this is the default behaviour for new projects, and if you would like the old behaviour you will need to change your project properties.

Types can be nested, to some extent.  Any kind of type can be nested inside a class or a struct, but not inside other kinds of type.  The full name of a nested type incorporates the name of the types it is nested inside: for example, if a class called `Thing` contained a nested type called `Egg`, the full name of the latter is `Thing.Egg` and the fully-qualified name of it is `Thing.Egg` prefixed by `Thing`'s namespace.

Each built-in type which can be instantiated has a fully-qualified name (which is the same across all CLI languages) and a C# keyword which can be used as a synonym; the convention is to use the keyword in code where possible.  The built-in types and their keywords are:

| Type           | Keyword | Constant suffix | Notes                                               |
| -------------- | ------- | --------------- | --------------------------------------------------- |
| System.Object  | object  |                 | Reference type.                                     |
| System.String  | string  |                 | Reference type.                                     |
| System.Boolean | bool    |                 |                                                     |
| System.Byte    | byte    |                 | Unsigned.                                           |
| System.SByte   | sbyte   |                 | Signed.  Not a CLS type.                            |
| System.Char    | char    |                 |                                                     |
| System.Int16   | short   |                 | Signed.                                             |
| System.UInt16  | ushort  |                 | Unsigned.  Not a CLS type.                          |
| System.Int32   | int     |                 | Signed.                                             |
| System.UInt32  | uint    | `U`             | Unsigned.  Not a CLS type.                          |
| System.Int64   | long    | `L`             | Signed.                                             |
| System.UInt64  | ulong   | `UL`            | Unsigned.  Not a CLS type.                          |
| System.Decimal | decimal | `M`             | Fixed-precision.                                    |
| System.Single  | float   | `F`             | Single-precision floating point.                    |
| System.Double  | double  | `D`             | Double-precision floating point.                    |
| System.IntPtr  | nint    |                 | Native-precision signed integer.  Not a CLS type.   |
| System.UIntPtr | nuint   |                 | Native-precision unsigned integer.  Not a CLS type. |

Types in the above table are value types unless otherwise noted.

Some numeric types have "constant suffixes" listed in the table above, which can be used to specify that a numeric constant, in code, is of that specific type.  Unsuffixed numeric constants with a decimal point default to `double`; without a decimal point, they default to the first integral type of `int` or larger (sorted in order of maximum value) which will accommodate the value.  Note that this means that a constant that is larger than the maximum `int` value, but smaller than twice the maximum `int` value, will be treated as `uint`.  The suffixes are case-insensitive.

The `nint` and `nuint` types were introduced in C# 9, as fast integer types whose exact size may differ between systems.  You are guaranteed to be able to assign constants within the ranges of `int` and `uint` to `nint` and `nuint` respectively.

All of the built-in numeric types have `MinValue` and `MaxValue` members.  In most cases these are compile-time constant fields; for the `nint` and `nuint` types they are runtime properties.  The floating-point types also have `Epsilon` (the smallest value greater than zero that can be represented), `NaN`, `NegativeInfinity` and `PositiveInfinity` fields.

The `string` type is immutable, and all "string manipulation" code actually returns a new string.  There is more information on string manipulation below.

By convention the names of interface types always begin with a capital `I`; other types do not have any significant naming converntions that apply across every type of that kind.

#### Inheritance and interface implementation

All user-defined types in C# inherit from exactly one other type, referred to as its "base type" or (for classes) "superclass", but this can not always be a type chosen by the developer.  A type that inherits from a given type can be referred to as a "derived type".  All user-defined value types inherit from the built-in type `ValueType`; this cannot be changed and is not specified in code.  Classes can inherit from any accessible class that is not declared as `sealed`; if the programmer does not explicitly specify a superclass, the class will inherit from the built-in `object` class.  Record classes can inherit from any accessible record class that is not declared as `sealed`, and like classes, inherit from the built-in `object` class if no base record class is specified.  Interfaces can inherit from other interfaces, but this is optional.

Classes, records and structs may implement any number of interfaces.  A class, record or struct which inherits a given interface is also considered to be a derived type of the interface.  Records and record structs cannot implement interfaces which contain methods, because records cannot contain methods.

An object of a given type may be assigned to a variable of any base type&mdash;or, in other words, if you have a variable of a particular type, an object of any derived type may be assigned to it.  In this situation, the type of the variable is the base type, but its *specific type* is the type of the object it contains at the given point in time.  To give an example:

```
Thing baseVariable = null;
DerivedThing derivedVariable = new DerivedThing();

// The type of baseVariable is Thing, but this assignment changes its current specific type is DerivedThing
baseVariable = derivedVariable; 

// After this assignment, its type and specific type will be the same.
baseVariable = new Thing();
```

The syntax for specifying inheritance and interface implementation is to put a colon after the class or struct name, followed by the name of the base class (if present), followed by a comma-separated list of interfaces.  In the following example, the `HensEgg` class inherits from the `Egg` class and also implements the `IZygote` and `ICrackable` interfaces.

```
class HensEgg : Egg, IZygote, ICrackable
{
    // ...
}
```

It's normal and idiomatic for classes in C# to implement interfaces.  However, structs implementing interfaces is somewhat rarer.  There is a slight performance disadvantage when casting a struct to an interface type; moreover, subtle behavioural differences can be induced when you do so.  Both of these are down to the fact that structs are value types but interfaces are not, and therefore casting the former to the latter leads to boxing and unboxing code being generated.  The same applies to casting any value type to the `object` type.

#### Arrays

An array of any type can be created by adding square brackets to the type name.  For creating multi-level arrays, C# distinguishes between *multidimensional arrays* which are a single multidimensional data structure, and *jagged arrays*, which consist of an a one-dimensional array whose elements are themselves arrays.

```
// One-dimensional array
int a[];

// Two-dimensional array
int b[,];

// Two-level jagged array
int c[][];
```

Arrays are reference types, derived from `System.Array`, and because of this jagged arrays do behave differently to multidimensional arrays.  With the former, you can only guarantee that the "first dimension" of the array actually exists.  If you declare an array of arrays of `int` values, for example, without initialising any data, you will initially create a one-dimensional array of `null` values, where each item in the array could potentially store a reference to an array of `int`s but currently stores nothing.   By contrast, if you create a multidimensional array of `int` values, each item in every dimension of the array is guaranteed to have a value because `int` is a value type.

The elements of arrays are accessed using the "indexer access operator", `array[i]`.  This operator is also used to access elements of other types by numbered or named index, such as lists or dictionaries.  You can create types which use this operator with any type as its operand type, but this is dissuaded.

Arrays are created with the `new` operator, and must have their size specified at creation time.  For jagged arrays, this only applies to the top level of the array.  They can also be initialised at creation time; if you initialise them, their size and type can be implied from the initialisation data.

```
int a[] = new int[5];
int b[] = new int[] { 2, 4, 6, 8, 10};

int c[,] = new int [5, 2];
int d[,] = new int[,] { {2, 3}, {4, 9}, {6, 12}, {8, 15}, {10, 18} };

int e[][] = new int[5][]; // Note that the length of the second level is not specified.
int f[][] = new int[][] { new [] {2, 3}, new [] {4, 9, 5}, new [] {8, 15}, new [] {10, 18, 25} };
```

Jagged arrays also have a shorthand initialiser form which omits the outermost `new` operator.

```
int f[][] = { new [] {2, 3}, new [] {4, 9, 5}, new [] {8, 15}, new [] {10, 18, 25} };
```

Array resizing is not implicit, but one-dimensional arrays can be resized with the `Array.Resize()` method.

An array of type `T` implements the `IEnumerable<T>` interface and can be assigned to the `ICollection<T>` and `IList<T>` interfaces, but because arrays are not implictly resizable, any methods of those interfaces which would cause a resize will throw an exception when called, for example `Add()` and `Clear()`.  The  following code will compile, but will fail at runtime:

```
int[] data = new[] { 1, 2, 3 };
IList<int> listData = data;
listData.Add(4);
```

#### Generic types

Reference types can be "generic types", that is, types that depend on other undefined types, known as "type parameters".  Such types are referred to using type parameters in angle brackets; a commonly-used example is `System.Collections.Generic.List<T>`.  When a variable of the generic type is declared, or when an instance of the type is instantiated, a type has to be provided for all type parameters; for example `List<string>`.  When a new generic type is defined, various restrictions can be applied to its type parameters if necessary.  These are introduced by the `where` keyword when defining the type; here are a couple of examples.

```
class Nest<T> where T : Egg // T must be an Egg or a class derived from it.
```

```
class Nest<T> where T : class // T must be a class type
```

When declaring generic interfaces, method parameters and return types, type parameters can be specified as covariant or contravariant.  See below for more information.

#### Assemblies

In any CLI language, the executable code is stored in one or more "assemblies"; the code in an assembly can also call code in other assemblies, called "referenced assemblies".  References are resolved at both compile time and run time.  At compile time this resolution can be done against a "reference assembly" which only contains stubs of the public content of an assembly; at run time the full assembly is needed.  Code can use "reflection" to interrogate the VES at runtime to determine which assemblies have been loaded, what types those assemblies contain, and to read all of the metadata associated with those types and the compiled code that implements them.  The VES does not have to load referenced assemblies, however, until they are required, so this sort of reflection-based interrogation does not necessarily return details about all assemblies referenced by the running assembly.

Each assembly consists of at least one "module file" which contains code, and can also include additional module files or resource files.  Assemblies can be "strong-named", which means they are crytographically signed, but this is optional.

### Some common conventions

- Traditionally, the entry point for an executable project is the method `Program.Main()`.  In general, when you create an executable project, the entry point will be created for you as part of your boilerplate code.
- A Visual Studio project normally corresponds to a single assembly.
- Within each assembly, all types should belong to the same namespace or to namespaces hierarchically under that "root namespace".
- The root namespace of an assembly should be the same as the name of the project, and the same as the filename of the assembly's main module file.
- The filesystem hierarchy of the source code in a project should mirror the hierarchy of namespaces in the assembly.
- Identifiers generally use CamelCase.
- Type names generally begin with a capital letter, as do non-private member names and all method names.
- In general there should usually be a 1 to 1 mapping between non-nested types and source code files.
- The name of each source code file should be the same as the name of the type it defines.
- The above two conventions are modified slightly if a class consists of a mixture of hand-written and generated code, or if classes are very tightly coupled.

It is possible to split a class between multiple files using the `partial` keyword.  Because of this, if a class consists of a mixture of hand-written and generated code, it is common to have two files, one for each kind of code.  This is a common pattern seen in Windows Forms development, for example: a class called `MainForm` will be split across the files `MainForm.cs` and `MainForm.designer.cs`, the latter containing the Visual Studio-generated form layout and event binding code, and the former containing the developer-written event handler methods and any other code.

It can also be acceptable to include more than one class in a file if the two are tightly coupled.  For example, there is a standard interface `IComparer<T>` which can define a class to be used to compare the lexicographical ordering of two instances of another class.  If you had defined a `Thing` class, and wanted to also define a `ThingComparer` class that implemented `IComparer<Thing>`, you may consider it acceptable to put them in the same file because of their tightly-coupled nature.  Similarly, if you define an enum type purely to use it as the parameter type of a single method in a single class, you may want to define it as a non-nested type in the same file as the class, or you may want to define it as a nested type within the class.

From C# 10 (.NET 6) onwards, it is possible to have one file in a project start with "top-level statements", statements which are defined outside any method, class or namespace.  If such a file exists, its top-level statements must be before any type definitions in the file, and the start of this file is the entry point.  The .NET 6 template for a console app creates a top-level statement file instead of a `Program` class.  You will get a compilation error if a project contains both a top-level statement file and a `Program.Main()` method. 

### An outline of class terminology

Most of the code in a C# program is generally in classes, given the object-oriented nature of the language.  This section gives an overview of some of their features, without delving into excessive detail or syntax.

A class type contains "members", each member being one of the following:
- Fields
- Properties
- Methods
- Events

The class itself and all of its members have an access level, which determines its visibility to other code.

*Fields* are essentially class-level variables.  Each field has a type, and can optionally have an initialiser.

*Methods* are where the majority of code goes; they are functions attached to classes.  Each method has a return type (which can be `void` if the method returns nothing) and a parameter list, which can be empty, and can be of variable length (with certain restrictions).  Method names can be overloaded, by defining multiple methods with the same name but different parameter types.  Overloaded methods do not have to have the same return type, but in C# (like Java) it is not possible to have method overloads that *only* differ by return type.

Methods may be generic (have type parameters, in other words), even if their containing class doesn't have any type parameters.  Method parameters may have contravariant generic types, indicated by putting the keyword `in` before the type parameter, and method return types may have covariant generic types, indicated with the `out` keyword.

A lambda method is a kind of anonymous method that can be defined within an expression inside a class.  They are defined using the `=>` operator; the syntax for this will be discussed later.  Lambda methods were introduced in C# 3.

*Properties* are, at the CIL level, just a method or a pair of methods linked by an identifier, which can then be called using the same syntax as that used to access a field&mdash;and in early versions of C# that is all they were.  In C# version 3 and above the compiler can automatically generate code that in earlier versions had to be added as boilerplate by the developer; version 7.0 introduced new syntax extensions that enable properties to be defined as lambda methods.

Each class has one or more "constructors", which are methods that are called when instantiating the class.  If the programmer does not define any constructors, the compiler implicitly inserts a constructor with no parameters, whose access level matches the access level of the class.

Each class can, optionally, have a "finalizer", which is similar to a C++ destructor and is defined with a similar syntax.  I'll talk about these more in the section on garbage collection and object lifecycles, but they are relatively rarely used.

### Access modifiers

Every type and member of a type has an access level, whether specified explicitly or not.  If it is specified explicitly, this is done by adding an *access modifier* where the type or member is defined.

The allowed access modifiers (and combinations) and their meaning are:

| Modifier(s)          | Allowed at top level | Meaning                                                                                |
| -------------------- | -------------------- | -------------------------------------------------------------------------------------- |
| `public`             | Yes                  | Unrestricted access from any code.                                                     |
| `protected`          | No                   | Can only be accessed from the same type or derived types.                              |
| `internal`           | Yes                  | Can only be accessed from the same assembly and potentially specific other assemblies. |
| `protected internal` | No                   | Equivalent to `protected` OR `internal`.                                               |
| `private`            | No                   | Can only be accessed from the same type.                                               |
| `private protected`  | No                   | Can only be accessed from the same type, or derived types within the same assembly.    |

The `private protected` modifier combination was introduced in C# 7.2.

The only access modifiers that can be used on top-level types&mdash;any non-nested types in other words&mdash;are `public` and `internal`.  The default access level for top-level types is `internal`; it is considered good practice to specify it explicitly, although generated code often does not do so.

The default access level for members of classes and structs, including that of nested types, is `private`.  You can change the access level of members of structs, and of all members of classes apart from finalizers.  You cannot set the access level of a class finalizer.  The default access level for members of records depends on how the record is defined.

The allowable access levels of the members of a struct are `public`, `internal` or `private`.  Because structs cannot have other types derived from them, the `protected` access modifier would be meaningless.

A type member's access level cannot have wider access than the access level of the containing type.  In other words, if a type is `internal`, none of its members can be `public`.  Similarly, if a method is `public` its parameters' types must also all be `public`.  However, the access level of a nested type can be wider than its containing type.

If a property has two accessor methods, they do not have to have the same access level.

You cannot modify the access level of the values of enum types.  Officially they always default to `public`, but in practice they effectively default to the access level of the containing type (which has to be either `public` or `internal`).  Before C# 8, you could not modify the access level of interface members, and when a class or struct implemented an interface the implementing members' access level had to match the type's access level.

Access levels are checked at both compile time and run time, but do not guarantee secrecy.  Under default circumstances, if a piece of code has a reference to an object, it can examine any member of that object and call any method on it, whatever the access level, using reflection techniques.  This can be restricted using the Code Access Security feature of .NET Framework, but this is rarely used and is not available in .NET (or .NET Core).

As I said in the table above, `internal` types or members can only be accessed from the same assembly or from specific assemblies (putting aside the fact that they can also be accessed from any code using reflection as per the previous paragraph, too).  It is possible to add code to an assembly so that other specifically-listed assemblies can access all of the internal code within the first assembly.  If the first assembly is strong-named, the accessing assemblies also need to be strong-named.

### More detail on class syntax

Classes are defined as follows:

```
namespace MyExample
{
    public class Egg
    {
        // ...
    }
}
```

The above defines a class called `Egg`, fully-qualified name `MyExample.Egg`, which has the `public` access level.  Because it doesn't specify its base class, its base class is `object`.

#### Fields

Fields are class-level variables, and can be declared with just a type and an identifier.  They can also include an initialiser.  They can be declared `readonly`, which means they can only be set in an initialiser or a constructor.  All fields without an initialiser will be initialised to the default value of their type.

```
namespace MyExample
{
    public class Egg
    {
        double _length;                            // No initialiser.
        double _width = 0d;                        // Initialiser
        double readonly _materialThickness = 0.05; // Cannot be changed.
    }
}
```

All of the fields in the above example will be `private` as they have no access modifier.

When declaring any variable or field, you can use the `var` keyword instead of a type if the compiler can deduce the type from the initialisation expression.  Some developers would consider it bad form to use `var` if the type is not immediately human-readable&mdash;if, for example, the initialisation expression is a method call.  `var` cannot be used to declare a non-initialised field.

```
namespace MyExample
{
    public class Egg
    {
        var _length = 0d;  // _length is of type double, due to the constant initialiser.
        var _width;        // Syntax error! There is no way to infer the type of _width.
    }
}
```

Generally speaking, the naming convention is for non-private field names begin with a capital letter.  A common convention for private field names is for them to begin with an underscore.

Instance fields are accessed using the `.` operator on an instance of the containing type.

```
namespace MyExample
{
    public class Egg
    {
        public double Width;
    }
}

// If you have a variable called egg of type Egg, the Width field is accessed as egg.Width
```

#### Constructors

The above example will have an implicit parameterless constructor inserted by the compiler, with the same access level as the class itself.  We can however define any number of constructors explicitly, as long as they have different parameter signatures.  A constructor's name is the same as the class name, and its return type is implicit.  If any constructors are explicitly defined, the implicit parameterless constructor is not created.

```
namespace MyExample
{
    public class Egg
    {
        double readonly _length;
        double readonly _width;

        public Egg(double size)
        {
            _length = _width = size;
        }

        public Egg(double width, double length)
        {
            _width = width;
            _length = length;
        }
    }
}
```

Constructor parameter names conventionally begin with a lower case letter for classes and structs, and an upper case letter for records (of either type); the reason behind this will be explained under the section on records below.

From C# 7 onwards, constructors that contain a single expression statement can be defined using the `=>` operator instead of with a block.

```
namespace MyExample
{
    public class Egg
    {
        double _length;
        double _width;

        public Egg(double size) => _length = _width = size;

        public Egg(double width, double length)
        {
            _width = width;
            _length = length;
        }
    }
}
```

There are various things that can be done with constructor parameters, such as giving them default values or calling them by name, which also apply to method parameters and are described below under "Methods".

If we had not explicitly set the access level of the class to `public` (or had explicitly set it to `internal`, which would have been equivalent) we would not be able to declare our constructors as public, due to the rules set out previously in the Access Modifiers section.  The following code will not compile:

```
namespace MyExample
{
    internal class Egg
    {
        double _length;
        double _width;

        public Egg(double size)
        {
            _length = _width = size;
        }
    }
}
```

Constructors are called using the `new` operator.

```
Egg egg1 = new Egg();
Egg egg2 = new Egg(6d, 4f);
```

From C# 9.0 onwards, you can omit the type name when calling the constructor, if the compiler can see what type is being assigned to.

```
Egg egg1 = new();        // These are both equivalent to the above in C# 9.0
Egg egg2 = new(6d, 4f);  // but fail in older versions.

var egg3 = new();        // Syntax error! We haven't specified a type at all.
```

A class's constructors are not inherited from its ancestors.  The only constructors available for a class are those defined in the class itself (or the parameterless constructor if there are none defined), not those defined in its base classes.  However, when you call a constructor, it always calls a constructor of the immediate superclass first, before it executes.  This constructor will then call its superclass constructor before it executes, and so on.  The end effect is that a chain of constructors will execute, starting with the constructor of `object` (which is parameterless) and then executing the constructors of derived types in order, until the constructor of the specific type is executed last.

By default, each constructor calls its superclass's parameterless constructor, with an implicit call inserted by the compiler.  However, this leads to a compilation error if the superclass does not have a parameterless constructor.  In other words, the following code is not allowed:

```
namespace MyExample
{
    // This class does not have a parameterless constructor, because we
    // have declared one with parameters.
    public class Egg
    {
        double _length;
        double _width;

        public Egg(double width, double length)
        {
            _width = width;
            _length = length;
        }
    }

    public class ChocolateEgg : Egg
    {
        // This code will not compile, because it would need to make an implicit call to the
        // nonexistent parameterless Egg constructor.
        public ChocolateEgg() { }
    }
}
```

However, you can specify that the derived class's constructor(s) should call a parameterised constructor in the immediate superclass, using the `base` keyword.  This is required if the superclass has no parameterless constructor, and avoids the compilation error in the previous example.

```
namespace MyExample
{
    public class Egg
    {
        double _length;
        double _width;

        public Egg(double width, double length)
        {
            _width = width;
            _length = length;
            Console.WriteLine("Egg constructor completed.");
        }
    }

    public class ChocolateEgg : Egg
    {
        public ChocolateEgg() : base(3d, 3d)
        {
            Console.WriteLine("ChocolateEgg parameterless constructor completed.");
        }

        public ChocolateEgg(double width, double length) : base(width, length)
        {
            Console.WriteLine("ChocolateEgg(double, double) constructor completed.");
        }
    }
}
```

With the above example, calling `new ChocolateEgg()` will produce the following output:

```
Egg constructor completed
ChocolateEgg parameterless constructor completed.
```

If a class has multiple constructors, you can instead specify that a constructor should call another constructor in the same class before executing, using the `this` keyword.

```
namespace MyExample
{
    public class Egg
    {
        double _length;
        double _width;

        public Egg(double width, double length)
        {
            _width = width;
            _length = length;
            Console.WriteLine("Egg constructor completed");
        }
    }

    public class ChocolateEgg : Egg
    {
        public ChocolateEgg() : this(3d, 3d)
        {
            Console.WriteLine("ChocolateEgg parameterless constructor completed");
        }

        public ChocolateEgg(double width, double length) : base(width, length)
        {
            Console.WriteLine("ChocolateEgg(double, double) constructor completed.");
        }
    }
}
```

In the above example, calling `new ChocolateEgg()` will produce the following output:

```
Egg constructor completed
ChocolateEgg(double, double) constructor completed.
ChocolateEgg parameterless constructor completed.
```

#### Methods

Methods are defined similarly to constructors, but they have a return type and a method name.  If a method returns nothing, its return type must be defined as `void`.  By convention all method names begin with a capital letter, although this is sometimes ignored by generated boilerplate code, for example for event handlers.  Traditionally, all methods were defined as "block methods" with a block of code&mdash;which in the next example happens to be empty.

```
namespace MyExample
{
    public class Egg
    {
        public void Crack()
        {
            // method implementation
        }
    }
}
```

Variables declared inside a method are *local variables*.  They are private to the method, cannot be accessed outside the method and cannot retain their value between invocations&mdash;there is no direct equivalent in C# to a C static local variable.  By convention local variable names begin with a lower-case letter.

Within a method, the special keyword `this` is a reference to the current instance of the class.  This can be used to distinguish within a method or constructor to distinguish between class members, and local variables or method parameters, which have the same names: if `thing` is the name of both a class field or property and a method parameter or local variable, then unqualified `thing` is the local variable and `this.thing` is the class member.

Methods are called using the `.` operator.  In method calls, the parameters are enclosed in brackets.  One common idiomatic C# style is "fluent code", in which methods return either `this`, or an object representing some form of transformation of `this`.  This enables you to chain a sequence of method calls together in a single statement, without declaring variables to store intermediate results.  This style of programming is often used in LINQ code, explained below under "Iteration".

Control is allowed to drop out of the end of a `void` method, or you can return from an earlier place in the method body with a `return;` statement.  However, block methods with other return types in general must return with a `return` statement that includes an expression to provide the return value.  If you create a method that has a return type, and a possible code path through to the end of the method that does not return a value, you will get a compile-time error.  The method doesn't have to end with a `return` statement if the compiler can detect that the end of the method body is unreachable.

Like a constructor, the parameter list of a named method can contain any reasonable number of parameters, but must specify the type of each.

```
public int ExampleMethod(int x, int y, double twist)
{
    // ...
    return x;
}
```

From C# 6 onwards, methods that contain a single return statement can be defined using the `=>` operator instad of a block.  The `return` keyword does not need to be included; the method implicitly returns the value of the expression.

```
public int Double(int x) => x * 2;

// ...is the equivalent of:
public int DoubleBlock(int x)
{
    return x * 2;
}
```

Methods can be overloaded, by specifying methods that share a name but have different parameter types.  The names of the parameters and the return type of the method do not count as differences as far as method overloading resolution is concerned.  However, if a method is overloaded with different parameter types, the overloads may have different return types.

You can specify default values for some "optional" parameters, so long as all following parameters also have default values.  When you call a method with multiple optional parameters, you must specify all parameters up to and including the last parameter you want to specify

```
// This method can be called as ExampleMethod(x, y) or as ExampleMethod(x, y, twist)
public int ExampleMethod(int x, int y, double twist = 0d) { ... }

// This method can be called as ExampleMethod(), ExampleMethod(x), ExampleMethod(x, y) or as ExampleMethod(x, y, twist)
// but not, for example, as ExampleMethod(twist)
public int ExampleMethod(int x = 0, int y = 0, double twist = 0d) { ... }

// This is a syntax error!
public int ExampleMethod(int x, int y = 0, double twist) { ... }
```

Note that when you call a method using the default values for optional parameters, the values of the optional parameters are computed and hard-coded into your CIL bytecode, at the calling location, at compile time.  This has two significant implications.  Firstly, the values of the optional parameters must be compile-time constants&mdash;you can't use the value of a method call as a default parameter value, and most reference type parameters (other than `string`) cannot have default values.  The second significant implication is that, if you change a parameter's default value, the *calling* code needs to be recompiled for the change to take effect, not the code which was actually changed.  Because of this, if you publish an API that contains public methods with optional parameters, it is a bad idea to ever change the parameters' default values.

Methods can have genuinely variable numbers of parameters, so long as all of the variable parameters are of the same type, by making the final parameter an array type and declaring it using the `params` keyword.  If, say you declare the following method:

```
public void ExampleMethod(params int[] variableParams) { ... }
```

then it can be called with any number of integer parameters (including none), each of which can be accessed by position as a member of the `variableParams` array.

If the variable parameter type is an array of a reference type, it enables parameters of any derived type to be passed, but the only way to have a variable number of parameters of any type is by declaring the `params` parameter to be of type `object[]`.  You can use reflection-based type-sniffing techniques within the method to discover the specific types of the variable parameters; this is described under "Reflection" below.

Methods with variable parameters can also be called explicitly with an array.  The method declared above could be legally called in any of the following ways:

```
ExampleMethod();
ExampleMethod(1);
ExampleMethod(1, 2, 3, 7);
int[] values = new[] { 1, 3, 2, 7 };
ExampleMethod(values);
```

Beware that if you have an overloaded method and one of those overloads has a `params` parameter, then the compiler's overload resolution may sometimes produce results you didn't expect.  This is particularly the case if the method has an overload that takes an `IEnumerable<T>` or `IList<T>` parameter as well as a `params T[]` parameter, or if the method takes a `params object[]` parameter.  

The method-calling examples above show methods called using positional parameters; this is the most common calling code convention.  Methods can also be called using named parameters (with the parameters in any order) or by a mixture of named and positional parameters.  In the latter case, the supplied positional parameters must come first, starting with the first parameter to the method, and within the range of positional parameters supplied, none can be skipped.

```
public int ExampleMethod(int x, int y, double twist) { ... }

// can be called as ExampleMethod(1, 2, 7d) with fully positional parameters
// or as ExampleMethod(twist: 7d, x: 1, y: 2) with fully named parameters (they can be in any order)
// or as ExampleMethod(1, twist: 7d, y: 2) with a mixture of positional and named parameters, the latter in any order
// but *not* as ExampleMethod(twist: 7d, 1, 2) or as ExampleMethod(1, 7d, y: 2)
```

In some ways, the classification of types into reference types and value types reflects how method-calling appears to behave.  Technically, however, all parameters are passed by value; it is just that reference type variables consist of references to objects, so when the reference is passed by value it still refers to the same instance.  You can see this if you change the value of a reference-type parameter within a method to refer to a different instance: the reference in the calling code will still refer to the original instance.  

C# does, however, implement true pass-by-reference if the parameter is declared as `ref` in both the method definition and the calling code.  As you would expect, if the value of a by-value parameter is changed inside the method the change is not visible to the calling code, but if the value of a by-reference parameter is changed inside the method, it is.  `ref` parameters must be initialised before use, and they must be variables (either local variables or fields); they cannot be constants, properties or expressions.

Consider the following example methods

```
public void AttemptToIncrement(int i)
{
    i++;
}

public void IncrementByRef(ref int i)
{
    i++;
}
```

The first method is effectively pointless; the second method is still not exactly useful outside the bounds of example code, but it will actually do something!

```
int i = 0;
AttemptToIncrement(i);
Console.WriteLine(i);  // prints 0
IncrementByRef(ref i);
Console.WriteLine(i);  // prints 1

// Writing IncrementByRef(i) would give a compile-time error.

/* This code would also give a compile-time error, because i has not been initialised:

int i;
IncrementByRef(ref i);

*/
```

There are two other ways to pass parameters by reference: `out` and `in`.  `out` is for parameters that have not necessarily been initialised before the method call; the method *must* modify them before it completes.  `in` parameters, which were introduced in C# 7.2, are the reverse: they guarantee that the method will not modify the parameter.  They can be used as an optimisation to save copying a parameter which is treated as read-only inside a method, which can be significant if the parameter is a large struct or record struct.

Both `out` and `in` parameter modifiers are verified at compile-time: you will get a compilation error if a method with any `out` parameters has a code path which does not modify all of the parameters or tries to access any of them before they are modified; and you will also get a compilation error if a method tries to modify an `in` parameter, or if you call a method with an uninitialised variable as an `in` parameter.

Like `ref`, `out` must be specified both in the method declaration and at the calling site, and `out` parameters must be variables.  `in` must be specified in the method declaration, but does not by default have to be specified at the calling site.  Indeed, using `in` at the call site does constrain the compiler in ways which force the parameter to be a variable, in the same way as `ref`; not using `in` at the call site allows constants, expressions and properties to be used as `in` parameters.  In this case the compiler will generate code which passes a reference to an anonymous temporary variable.

Reference types can also be passed by reference, instead of passing the reference by value.  This is only really relevant in situations where a reference type parameter is reassigned.  Strictly speaking assignment is the only operation that can change the value of the reference that is stored in a reference type variable, as opposed to changing the fields and properties of the object the reference is to.  Passing a reference type by reference is most useful when dealing with string parameters.  `string` objects are an immutable reference type, so "changes to a string" actually create a new string object.  Strings and their specific behaviour are discussed further below, but it's worth remembering that if a developer is unfamiliar with C#, it is easy to write "string modification" code in which the modifications are not actually visible to the caller.

Passing parameters by value or by reference counts as a difference for method overloading, but passing parameters by reference in different ways does not.  In other words, this example works:

```
public class Example
{
    public void DoStuff(int a) { }

    public void DoStuff(ref int a) { }
}
```

but this example will not compile:

```
public class Example
{
    public void DoThings(in int x) { }

    public void DoThings(out int y) { }
}
```

If you are calling a method with an `out` parameter, but you do not need to set it beforehand or use its result afterwards, then (since C# 7.0) you can use a *discard variable* as the name of the variable in your call.  As mentioned above, the discard variable name is a single underscore.

For example, one common pattern for converting a number to a string is the `int.TryParse()` method.  It takes two parameters: the first is a string, and the second is an `out int` parameter.  The normal way to call it is as follows:

```
string str = "83";
int x;
bool success = int.TryParse(str, out x); // success will be true, x will be 83.
str = "some words";
success = int.TryParse(str, out x); // success will be false, x will be 0.
```

However, if you just want to find out whether or not the string is a valid number, but do not care about its actual value, you could use a discard variable for the out parameter, like this:

```
string str = "37";
bool success = int.TryParse(str, out _); // success will be true.
```

Note that you do not declare `_`&mdash;indeed, this only works if `_` has not been declared as a variable (or anything else).  For compatibility with earlier code, if you declare `_` then it becomes a regular variable and is not the discard variable.

Various other modifiers can be applied to methods, but I will describe those in full later, after introducing the relevant concepts.

#### Properties

A property has a type and a name, and essentially (at the CLI level, and in C# 1) consists of an optional `get` method and an optional `set` or `init` method.  It is accessed like a field; access that would read from a field instead calls the `get` method, and access that would write to a field calls the `set` method.  The `get` method has no parameters and its return type matches the type of the property; the `set` method returns `void` and has one parameter, named `value`, which also matches the type of the property.  Neither method has to have a method signature declared in the usual way.  By convention, property names begin with a capital letter whatever their access level.  Properties that only have a `get` method are read-only; properties that only have a `set` method, which are much rarer, are write-only.  The `init` accessor method (introduced in C# 9) behaves like a `set` accessor, but can only be called from the containing type's constructor, giving the property similar behaviour to a `readonly` field.

Originally in C# 1, that was all there was to say about properties; actually *using* properties required the developer to insert boilerplate "backing fields" and methods.  You still might see this style of C# code in the wild:

```
public class Thing
{
    private bool _theft;

    public bool Theft
    {
        get { return _theft; }
        set { _theft = value; }
    }
}
```

In the above the `Thing` class has a public property, `Theft`, with a private backing field `_theft` and boilerplate methods for storing and retrieving the property value to and from the backing field.

Since C# v3, the compiler has been able to insert the boilerplate code for you, so the above example might have been updated to the following:

```
public class Thing
{
    public bool Theft { get; set; }
}
```

The above will compile to very similar CIL code to the previous example, but the backing field is now anonymous, and inaccesible to other code without going through the property itself.  This is referred to as an *auto-implemented property*.  Because the backing field is inaccessible, you can't have a property that is only half auto-implemented: if one of the methods is auto-implemented, the other must be too.  From C# 9 onwards, you can create a read-only auto-implemented property by declaring it to have an `init` accessor instead of a `set` accessor.

```
public class Thing
{
    public bool Theft { get; init; }
}
```

Properties can potentially be more complex than this, and can potentially include any sort of logic in their get and set methods, as in this example:

```
public class Thing
{
    public bool Theft { get; set; }

    public bool Murder { get; set; }

    public bool Illegality
    {
        get
        {
            return Theft || Murder;
        }
    }
}
```

In this example, `Theft` and `Murder` are both straightforward mutable auto-implemented Boolean properties; `Illegality` is a read-only property that will be `true` if either theft or murder have occurred.

Bear in mind, when creating a property that has complex logic, that it is probably a bad idea to create a property that does not behave in an intuitive and straightforward way to a developer who hasn't seen its implementation.  Particularly, you should generally be able to expect that if you store a value or object in a read-write property, then retrieve its value, you will (barring external events) get the same value or object back again.

From C# v6, you have been able to implement read-only properties as lambda expressions.  This enables you to rewrite the previous example in the following form, which does the same thing but is somewhat more terse:

```
public class Thing
{
    public bool Theft { get; set; }

    public bool Murder { get; set; }

    public bool Illegality => Theft || Murder;
}
```

Note that this syntax, whilst shorter, can in some circumstances make it harder to spot that this is a read-only property, as the `get` keyword has disappeared.  The syntactic difference between a read-only property and a parameterless method here is solely that the method would have `()` after its name. 

C# 7.0 and up allow you to implement read-write or write-only properties with lambda expressions in a similar way.  The earlier example with private backing variable can now be written in the following style:

```
public class Thing
{
    private bool _theft;

    public bool Theft
    {
        get => _theft;
        set => _theft = value;
    }
}
```

This can be useful for read-write properties that do require relatively simple additional logic.

All of the properties in the examples above have had the same access level for both `get` and `set` methods, set on the property itself.  It is however possible for either accessor method to have a more restrictive access level than the other, by declaring an access level on the accessor itself.  The most common use of this is to make an auto-implemented property effectively read-only to code in other classes, by giving it a private `set` method.  The following two examples are functionally identical in normal code:

```
public class Thing
{
    private bool _theft;

    public bool Theft => _theft;
}
```

```
public class Thing
{
    public bool Theft { get; private set; }
}
```

In the first example `Theft` is a read-only property with a private backing field; in the second, it is an auto-implemented read-write property with a private `set` method.  In both cases the property value can only be changed in the normal way by code within the `Thing` class; in the first, by changing the backing field, and in the second by changing the property value directly.  As was briefly mentioned above in the section on access levels, in both cases the value can be changed by reflective code; in that situation, slightly different code would be needed between the two cases.  Before C# 9 this pattern was commonly used to implement something close to immutable properties, by making the set accessor private and trusting the other code within the same type not to call it; however, the `init` accessor now gives us true immutable properties.

You can set properties immediately after calling a constructor using "brace initialiser syntax", also called "object initialiser syntax.  The following code:

```
Thing theThing = new Thing() { Theft = true };
```

is partially identical to:

```
Thing theThing = new Thing();
theThing.Theft = true;
```

The big difference here is that if `Theft` is defined with an `init` accessor, brace initialiser syntax will work but the second form will not.  If `Theft` is defined with a private `set` or `init` accessor, neither of the above forms will work unless this code is inside the `Thing` type. 

Where the constructor is parameterless, the constructor parentheses can be omitted when using brace initialiser syntax.  The first example above could also be written `Thing theThing = new Thing { Theft = true };`

From C# 9 onwards you can alternatively omit the type name and write `Thing theThing = new() { Theft = true };`.  However, you can't omit *both* parentheses and type name at the same time&mdash;this would clash with the syntax for an anonymous type declaration, which is described below.

Since C# 6, you can also set the default value of an auto-implemented property when it is declared:

```
public class Thing
{
    public bool Theft { get; set; } = false;
}
```

This pattern can also be used with properties that have an `init` accessor.

#### Hidden, virtual and overridden methods and properties

When one class inherits from another, all members of its base classes that are not private (either explicitly or implicitly) are accessible in the derived class.  Private fields, properties and methods of the base classes are not accessible in the derived class.

In code elsewhere, all of the accessible fields, properties and methods of the base classes are also accessible in any derived class.

In both of the above cases, by default, the implementation remains in the base class.

A derived class can implement a method with the same name as a method in a base class, if it has different parameter types, just as methods within a class can be overloaded.

If a derived class implements a method with the same name and type signature as a method in a base class, then by default the method in the base class is *hidden*.  The compiler will give a warning that this may not be what you intend, unless you use the `new` modifier on the derived class method.  When hidden methods are called, the exact implementation that gets called depends on the type of the reference through which the call is made, and not on the specific type of the object.  Consider the following classes:

```
public class Egg
{
    public void CrackShell()
    {
        Console.WriteLine("Shell smashed");
    }
}

public class ChocolateEgg : Egg
{
    // The new modifier tells the compiler we definitely meant to do this,
    // and hide the Egg.CrackShell() method.
    public new void CrackShell() 
    {
        Console.WriteLine("Chocolate cracked");
    }
}
```

The following code fragment:

```
ChocolateEgg choco = new ChocolateEgg();
choco.CrackShell();
Egg egg = choco;
egg.CrackShell();
```

would print

```
Chocolate cracked
Shell smashed
```

Note that when the `ChocolateEgg` instance is assigned to a variable whose type is its base class, and then a method call is made through that variable, the base class implementation of `CrackShell()` is called.

If a method in a base class is declared as `virtual`, then a method with the same name and type signature in a derived class will *override* the base method, instead of hiding it.  If we modify the previous example to make the `CrackShell()` method virtual:

```
public class Egg
{
    public virtual void CrackShell()
    {
        Console.WriteLine("Shell smashed");
    }
}

public class ChocolateEgg : Egg
{
    // Note we now need to use the override modifier instead of new.
    public override void CrackShell() 
    {
        Console.WriteLine("Chocolate cracked");
    }
}
```

then the same code fragment as before would produce the following fragment:

```
Chocolate cracked
Chocolate cracked
```

This time, the method invocation always calls the implementation in the specific type, whatever the type of the reference it is called through.  With non-virtual method calls, (called "static invocation" in the CLI standard, but this isn't the same thing as calling a C# static method), the implementation to be called is fixed at compile-time and written into the CIL code.  With virtual invocation, the implementation to be called is determined by the VES at run-time.  CIL actually has different bytecode instructions for static and virtual calls.

Like methods, properties can also be declared `virtual`, and virtual properties can be overridden using `override`, instead of being hidden.  Likewise, properties can hide the properties of base classes with the `new` modifier keyword.  If a read-write property is virtual, you have to declare the entire property to be virtual; you cannot, for example, have a virtual `get` method and a non-virtual `set` method.  The behaviour of hidden or overridden properties is exactly the same as the behaviour of hidden or overridden methods.  A property that hides another can have a different type, access level and/or different accessors to the hidden property, but a property that overrides another must have the same type, access level and accessors as the one it is overriding.

You cannot declare a private method or property to be virtual; this would be meaningless. Because a private member can't be accessed from a derived class, it can't be overridden.  If you try to, you will get a compiler error.

You can't use `override` if the method you are trying to override isn't declared as `virtual` (or `override`); this is also a compile-time error.

Methods and properties that are modified with `override` implicitly inherit their "virtual-ness"; if you try to declare them as `virtual override` you will get a compiler error on the grounds that the `virtual` modifier is superfluous.  If you want to change this, see below under "sealed classes and members".

C#'s behaviour, in this area, differs notably from Java.  In Java, all methods are virtual in the C# sense, and any derived class can override methods from its base classes without requiring any additional keywords to be added compared to a non-overriding method.  C# (and other CLI languages), by comparison, allow the developer of the base classes to decide whether or not third parties should be able to override the behaviour of those classes' methods on a method-by-method level.

#### Sealed classes and members

When a class is declared as `sealed`, you can't derive any other classes from it.  If you declare any protected members inside a sealed class, you'll get a compiler warning, as in that context `protected` has the same effective meaning as `private` (and `protected internal` the same meaning as `internal`).

An `override` member of a non-sealed class can also be declared as sealed.  In this context, `sealed override` "unvirtualises" a member&mdash;in other words, it prevents any further-derived types from overriding the member.  `sealed override` is effectively a form of `override` that does *not* imply `virtual`.

#### Abstract classes

If a class is declared as `abstract`, it can never be instantiated.  The `abstract` modifier is placed after the access modifier (if there is one) but before the `class` keyword.

```
public abstract class YouCantNewThis
{
    // ...
}
```

Abstract classes can contain abstract methods, which are methods that have a declaration but no method body.  The declaration ends with a semicolon.

```
public abstract class YouCantNewThis
{
    public abstract void ExampleMethod(); 
    // Note no code here!
}
```

Abstract classes can also contain abstract properties.  In their declaration, they resemble an auto-implemented property, because like an auto-implemented property their get and set methods (where present) have no body.

```
public abstract class YouCantNewThis
{
    public abstract string ReadWriteProperty { get; set; }
    public abstract int ReadOnlyProperty { get; }
}
```

However this is really just a coincidence of syntax; when an abstract property is overridden, it can be implemented automatically or manually.

Abstract methods and properties can *only* occur inside abstract classes; if you try to use them in a concrete class, you will get an error.  Abstract methods and properties are implicitly virtual, so if you try to declare them as `abstract virtual` you will also get an error.  However, the members of an abstract class don't *have* to be abstract themselves.

When you derive a concrete class from an abstract class, you must provide an implementation for all abstract properties and methods in the base class, and they must be declared `override`.  The same rules apply as when overriding virtual members of a concrete class: access levels, parameter types, return types and property accessors must all match.

Inheriting an abstract class and overriding its virtual methods and properties is similar in principle to implementing an interface.  However, there are three key differences:
- Members that are defined in order to implement an interface are not declared as `override` and are not implicitly `virtual`.
- Members that are defined in order to implement an interface must the same access level as their containing type, whereas members that are defined in order to implement an abstract class must have the same access level as the abstract member.
- A property of an interface with only a `get` accessor may be implemented by a property with `get` and `set` or `get` and `init` accessors, whereas an abstract property can only be implemented by a property with matching accessors.

#### Static members and classes

Static members of classes are accessed through the class name, and are not associated with an instance of the class.  In other words, given the following class:

```
public class Egg
{
    public static string Colour { get; set; }   // Not actually a very good example
}
```

...then the `Colour` property is accessed as `Egg.Colour`, and not accessed through an instance.  The same property applies to the class "across the board" within the same process, which is why (as the comment says) this is not actually a very good example, unless you are happy for all your eggs to be the same colour!

Static fields can have initialisers, as the following:

```
public class Egg
{
    private static string _colour = "Blue";
}
```

Static initialiser code is gathered up into a special method created by the compiler, called the class constructor, class initialiser or type constructor, which is guaranteed to be run before any instance constructor of that class is called.  The exact point at which a class constructor is called is not defined beyond this, and historically has varied according to implementation, so don't rely on it.  Don't, for example, rely on the precise behaviour of code like this:

```
public class Processor
{
    // This could be anywhere between the program start time and the time the first instance of this
    // class was created, depending on runtime implementation.
    public static readonly DateTime ApproxProgramStartTime = DateTime.Now;
}
```

You can't call the class initialiser yourself, but you can access its code and metadata via reflection like any other method.  You can't define your own class initialiser.

Methods (other than constructors) can be declared as static.  Inside a static method you can't use the `this` reference, or access any non-static members with an implied `this` reference.  Static methods can't be declared as `virtual` or `abstract`, so also can't be overridden; however, a static method can hide an inherited instance method or vice-versa.

A class can be declared as a static class if all of its members are static.  By implication, a static class can't have any constructors.  Static classes are implicitly `sealed`, so can't be inherited.  Even though a static class cannot contain `abstract` members, static classes are themselves implicitly `abstract`, as in CIL this is the flag used to mark a class as non-instantiable.

#### Extension methods

When developing, you may find that you wish you could extend the API of a type when you cannot.  Maybe it's something you don't control, maybe it's something that is `sealed`, maybe it's a type that can't have its own member methods.  You can partially get around this by writing *extension methods*.  These are static methods that can be called as if they were instance methods of any arbitrary type, even one that you do not control.

An extension method is a static method, in a static class, with at least one parameter, where the first parameter is declared with the `this` modifier.  It can be called as a normal static method; or, it can be called as if it were an instance method of any instance or value of the first parameter's type (including if the value is `null` for types that allow that).  The `this` parameter can be of any type, including things such as `enum` types that cannot normally have methods.

Imagine, for example, that you wish the `string` type had a `ToAlternateCase()` method that changed characters alternately to upper then lower case.  The `string` type is sealed, so you can't subclass it, and in any case you'd then have to replace `string` with your custom subclass in anything that used strings where you might want to use your new method.  Creating an extension method, by comparison, is straightforward.  One possible implementation would be as follows:

```
public static class StringExtensions
{
    public static string ToAlternateCase(this string str)
    {
        if (str is null)
        {
            throw new ArgumentNullException(nameof(str));
        }
        char[] letters = new char[str.Length];
        for (int i = 0; i < str.Length; ++i)
        {
            letters[i] = (i % 2 == 0) ? char.ToUpper(str[i]) : char.ToLower(str[i]);
        }
        return new string(letters);
    }
}
```

As long as the namespace of your `StringExtensions` class is in scope, or has been brought into scope with a `using` declaration, you can then call your new method as if it is a `string` instance method.

```
string example = "example";
string output = example.ToAlternateCase(); // output will be "ExAmPlE"
```

Giving classes containing extension methods names ending in `Extension` is a common convention, but not a requirement.  There is no rule that *all* of the methods in such a class have to be extension methods, only that the class must be static.

Note that when used our new method appears to have no parameters.  Instead, the reference it appears to be called on is passed in as the first parameter to the method.  If the extension method has more than one parameter, then all but the first are passed normally, with the first positional parameter in the call being the second positional parameter in the method signature and so on.

There is one major difference in behaviour between instance methods and extension methods, with respect to the caller.  If you call an instance method on a null reference, the VES will immediately throw a `NullReferenceException`.  However, because extension methods are really just a form of syntactic sugar that get substituted by a normal static method call in the CIL bytecode, calling an extension method on a null reference will not automatically throw an exception; you will just get a call to the method with `null` passed as the first parameter.  In many cases this will throw a `NullReferenceException` in the code, but it may not.  It can be worthwhile specifically checking for a null first parameter and throwing an exception manually at the start of an extension method, so that from the caller's point of view the behaviour will be straightforward and almost the same as an instance method, save for the exception being thrown from within the method rather than from its calling frame.  However, if all warnings are enabled, the compiler will give you a warning if you manually throw `NullReferenceException` at any point.  Throwing exceptions is discussed below.  In C# 9 and above you can avoid any issue by using a nullable context and declaring the `this` parameter non-nullable.

#### Const fields and readonly fields

The `const` and `readonly` modifiers are both used to declare read-only fields.  However, there are significant differences between them.

A `const` field is an implicitly-static compile time constant.  It has to be initialised with an expression that the compiler can compute at compile time.  This puts a number of limitations on `const` fields: they effectively have to be either value types or strings.  The value types must be "unmanaged value types"&mdash;value types that contain no references aside from references to strings&mdash;and their initialisers must either be literal values or expressions the compiler knows how to fold into a literal value.

Like method default values for parameters, const values are baked in to the CIL bytecode at the point of use when that code is compiled.  This means that one aspect of their behaviour is similar: if you change the source code value of a `const` field, both the code defining it and all the code using it has to be recompiled to pick up the new value.  In general, it is considered rather bad form to change the value of a `public const` field once it has been published.

You can use `readonly` fields a little more flexibly.  `readonly` can be used for both static and instance fields, and makes the field behave like a property with an `init` method.  Static fields must be initialised; instance fields must either be initialised or must be set in a constructor.  The value of a `readonly` field cannot be changed, once it has been set, but as it's set at runtime not compile time it can be any computable value or reference.

There are situations in which accessing a `readonly` field leads to a slight performance penalty, due to small amounts of boilerplate code that the compiler has to add in some situations to ensure that the field is not modified.  However, equally, declaring a field as `readonly` can lead to slight optimisations being applicable in some circumstances.  In general, Visual Studio will hint you to mark a field as `readonly` whenever it can be.  A good example of a candidate situation for `readonly` use is when dependencies are injected as constructor parameters, as these then probably should not change over the life of the object.

Note that although the value of a `readonly` field is fixed, it only grants you *shallow immutability*.  If it is a class or a struct, its members are not necessarily themselves immutable  This is particularly important to bear in mind in the case of `readonly` arrays: the `readonly` applies to the reference to the array, not to the individual values the array contains; it may be worth considering using an `ImmutableArray<T>` or similar class instead.  Shallow immutability applies to any `readonly` reference type or struct, unless that type is something immutable such as a `string`.

### Other aspects of syntax

#### Flow control

As stated above, most of the flow of control statements are very similar to other languages such as C, C++ or Java.  C# has a reasonably rich set of flow of control statements, and most of them behave in a reasonably intuitive way if you're familiar with other languages.  The main flow of control statements are `if`, `switch`, `while`, `for` and `foreach`.

In general, each flow of control statement accepts either a single statement or a block statement, and if it takes a condition, the condition will be an expression surrounded by parentheses.  For example the `if` statement's format is as follows:

```
if (condition)
    <statement-or-block>
else if (condition)
    <statement-or-block>
else
    <statement-or-block>
```

The `condition` expression is known as the "controlling expression", and has to either return a boolean value, or a type which defines the `true` operator (see "Expressions" below).

In the above, `<statement-or-block>` represents either a single statement or a block statement.  It is relatively common, however, for organisations' coding standards to mandate always using block statements in flow of control statements, even when the block only contains a single statement.  An `if` statement like the above can have any number of `else if` sections (including none), and zero or one `else` sections.

The `switch` statement is similar to other languages, but has a few restrictions.  In general it has the following format, with an expression which is evaluated, and control jumping to a matching `case` label:

```
switch (expression)
{
    case 1:
        // zero or more statements
        break;
    case 2:
    case 3:
        // zero or more statements
        return;
    default:
        // zero or more statements
        break;
}
```

Note that unlike C, C# does not allow you to fall through from one case to the next.  You can, though, as above, have two or more successive case labels with no statements separating them, to indicate that the same code should handle several expression values.  Normally the `break` statement is used to exit from the entire statement, but you can also use `return` to exit from the enclosing method, or indeed any other means of stopping execution from falling through which can be verified at compile-time, such as entering an infinite loop or exiting the VES entirely.  C# also provides a `default` label which execution will jump to if no `case` matches the expression value.

Historically (up to and including C# 6) the `switch` statement was quite restrictive.  The type of the expression had to be either `char`, `string`, an enum or an integer type (not necessarily `int`), and the case labels had to be either literal constants, enum values or `const` fields.  All of the case labels had to be mutually distinct.  From C# 7 onwards, this restriction has been removed.  The expression can be of almost any type (it can't have the type `null`, although its value can be null), and a case can either match any value of a specific type, or an arbitrary expression, using the syntax `case int i:` or `case int i when i > 10:` to match firstly any int, and secondly any int greater than 10.  With this syntax, note that a variable (`i` in the examples) is also declared, which can then be accessed in the following statements.

As in the C# 7 `switch`, case labels no longer have to be distinct from each other, the first matching `case` is the one that is executed.  If the compiler can prove that due to the ordering of cases, one or more will never be reached, you will get a compile-time error.  For example the following code:

```
switch (expression)
{
    case int i:
        Console.WriteLine(i);
        break;
    case int i when i > 10:
        Console.WriteLine("Big value!");
        break;
}
```

will not compile, because anything that could match the second case will clearly also match the first.  However if the two cases are swapped over, the code will compile.

From C# 8 onwards, `switch` can also be used as an expression rather than as a flow-of-control statement.  It follows the same basic principles as a switch statement: an expression is evaluated and the action taken by the `switch` is the first from a list of patterns matched by the result of the expression.  However the syntax is rather different, and there two other important differences:
- The action taken by a switch expression is the evaluation of a single expression, not the execution of a series of statements.
- All of the possible expressions to be evaluated&mdash;the "expression arms"&mdash;must have the same type.
- If none of the cases in a switch statement match the input expression, nothing happens.  If none of the patterns in a switch expression match the input expression, an exception is thrown.

As for a switch statement, if the compiler can prove that one of the expression arms can never be reached, you will get a compile-time error.  An example switch expression, based loosely on the previous switch statement, looks like this:

```
string result = expression switch
{
    10 => "At the limit.",
    int i when i > 10 => "Big value!",
    int i when i >= 0 => i.ToString(),
    _ => "Not allowed."
};
```

Hopefully you can see a few of the differences in syntax between a switch statement and a switch expression:
- The controlling expression is the left-hand operand of the `switch` operator, rather than being after the keyword.
- The controlling expression does not have to be in parentheses.
- The case keyword isn't used.
- `=>` is used instead of `:` between the pattern and the expression arm.
- Each expression arm ends with a comma, although this is optional for the final expression arm.
- Instead of the `default` label, the "discard pattern" `_` is used to match any value.
- Because this is an expression statement, it must end with a semicolon.

The remaining flow of control statements are all different forms of loop: `while` (and `do ... while`), `for` and `foreach`.  All of them support putting two statements within the loop: `break` to leave it entirely, and `continue` to jump straight to the next iteration of the loop, using the term "iteration" loosely as the exact behaviour differs slightly according to the kind of statement.

The `while` loop is probably the simplest:

```
while (expression)
{
    // ... loop body ...
}
```

If the controlling expression is `true`, the body of the loop will be executed.  The expression will then be evaluated again, and if `true` the body executed again, ad infinitum&mdash;the most straightforward to create an infinite loop in C# is arguably with `while (true) { ... }`.  As stated above, the `continue` statement will end the current iteration of the body and start the next evaluation of the loop expression, and the `break` statement will end the loop entirely and start executing the statement following the loop.

Note that like all these kinds of loop, the loop body may be either a block or a single statement; but it is common for coding standards to mandate using blocks.

The `do` loop is essentially a `while` loop with the loop expression moved to the end of the loop:

```
do 
{
    // ... loop body ...
} while (expression);
```

The only practical difference is that the loop body is always executed at least once, before the first evaluation of the loop expression.

The `for` loop is very similar to C and Java:

```
for (init; test; repeat)
{
    // ... loop body ...
}
```

The brackets contain three expressions separated by colons.  When the statement is reached, the `init` expression is evaluated; then if the `test` expression is `true`, the loop body is executed.  For subsequent iterations the `repeat` expression is evaluated, then if the `test` expression is true, the loop body is executed.  If the `test` expression is false, execution moves on to the following statement.

The `foreach` loop can be used to replace the `for` loop in many situations, where it is being used to iterate over the elements of some form of collection or enumeration.  It looks like this:

```
foreach (var loopVariable in enumeration)
{
    // ... loop body ...
}
```

To expand a little on the rather opaque example above: `foreach` and `in` are keywords.  `var loopVariable` declares a read-only variable which can be used within the loop, and `enumeration` is an expression of type `IEnumerable<T>`.  The loop variable must be of type `T`.  In the example I've declared the variable as `var`, which is always allowable here because the compiler will always know what `T` is, but it is not necessarily the best thing to do.  For one thing, coding standards sometimes rule against it in this situation, preferring an explicit type; for another, you may potentially want to declare the loop variable as a superclass of `T` instead.

The `IEnumerable<T>` interface is the base interface of all things iterable or enumerable.  The concepts behind this, and what you need to do to implement your own, are described below.  For now, all you need to know is that it is a type that can be used to access a set of items, all of the same type, sequentially.  When used in a foreach loop, the loop variable is set to the next item in the sequence, the loop body is executed; then the loop variable is set to the next item in the sequence and the loop body executed again, until there are no more items left in the sequence.

Note that the `IEnumerable<T>` interface has no concept of the length of its contents, or of the position of the current item within the sequence.  Implementations of it are also not necessarily repeatable.  If you are iterating over a sequence of knowable length and need to know the position of each item in the sequence within the loop body, it usually makes more sense to use a `for` loop or a LINQ method (see below for the latter).  Implementations of `IEnumerable<T>` will mark any active iterators as invalid if the underlying data is changed in certain ways, which can easily lead to runtime errors (for example, if you are iterating over a `List<T>` and add or remove list elements within the loop body).  It is best to treat the iterator, and anything underlying it, as immutable for the duration of the loop.

```
// This code will compile, but fail at runtime, because List<T>.Clear() 
// changes the list by removing all its elements, which breaks the active iterator.
List<int> data = new List<int>() { 1, 3, 4, 5 };
foreach (int i in data)
{
    if (i == 3)
    {
        data.Clear();
    }
}

// This code will succeed, because Array.Clear() only
// changes the value of each element to its default.
int[] data = new [] { 1, 3, 4, 5 };
foreach (int i in data)
{
    if (i == 3)
    {
        Array.Clear(data); // data is now 0, 0, 0, 0
    }
}
```

In the life of C# there has been one major breaking change, behaviourally, with respect to `foreach` loops.  In earlier versions of C# (up to and including version 4), when a `foreach` loop was being executed, the loop variable was conceptually the same variable across all iterations of the loop, set to a different value on each iteration.  With C# 5 and later, on each iteration of the loop, the loop variable is a different variable with the same name.  This might seem like semantic nitpicking, and in the vast majority of cases it is, but it leads to a difference in behaviour when loop variables are "captured" by using them within lambda expressions, as a reference to the loop variable can then legitimately "leak" out of the loop.  With older versions, that captured variable would then only ever have the value from the final loop iteration, whatever iteration it was captured on.  Lambdas are explained properly below; most code analysis tools will give you a warning when you have used a construct that might have behaved differently in older versions of the language.  The workaround&mdash;which will give the same behaviour on all language versions&mdash;is to assign the loop variable's value to a second variable within the loop body and refer to the value through that variable instead.  The change was made partly because of the large number of developers who found the original behaviour unintuitive.  Here's an example of some code that shows off this change in behaviour.

```
Action printAction = null;
foreach (int i in new int[] { 1, 2, 3, 4, 5 })
{
    if (i == 1)
    {
        printAction = () => Console.WriteLine(i);
    }
}
// This will print 1 in recent versions of .NET, 5 in older versions of .NET Framework.
printAction();
```

#### Exception handling

Like a number of other languages, C# supports structured exception handling.  The key points to this are fairly straightforward:

* At almost any time, code can *throw an exception* as a means of raising an error.
* Blocks of code can be defined as trapping any exceptions, or exceptions that match certain criteria
* If an exception is not trapped within a particular stack frame, it "bubbles up" to the calling frame
* If an exception bubbles all the way up to the top, the program exits.

Exceptions are classes; developers can create new exception types, but all exception types must inherit in some way from the `System.Exception` class.  In general, it is usual to instantiate a new exception object when throwing an exception, but it's not compulsary.  The `System.Exception` class contains various useful properties, such as `Message` to hold a human-readable error message, `InnerException` which I'll explain shortly, and `StackTrace` which holds a formatted string describing the site where the exception was thrown.  Code does not declare what types of exception it may potentially raise, and there is no way to do this other than documentation.

Exceptions are thrown using the `throw` statement:

```
throw new Exception("An error message");
```

The parameter sets the `Exception.Message` property; `Exception.StackTrace` is set automatically by the `throw` statement.  The `InnerException` property will be `null`.

In general it is best practice to throw the most suitable type of exception for the situation in question, if one exists, and create your own specific exception types if none of the system exceptions fit your needs.  The system exceptions fit common use cases such as checking method parameters:

```
public void Bake(CakeMixture mixture, CakeTin tin)
{
    if (mixture == null)
    {
        throw new ArgumentNullException(nameof(mixture));
    }
    if (tin.Size > this.Size)
    {
        throw new ArgumentOutOfRangeException(nameof(tin), tin.Size, "Tin too large for oven");
    }
}
```

If you have all warnings enabled, the compiler will give you a warning if you throw some exception types that are considered very broad, such as `Exception`, or are reserved for the VES to throw, such as `NullReferenceException` or `IndexOutOfRangeException`.

The `nameof` operator, which returns the name of its operand as a string constant, was introduced in C# 6.  Before that, the above code would have to be written as `throw new ArgumentException("mixture")`, and if you changed the name of the parameter in the code, you would have to remember to update your `throw` statements to match.

I described `throw` as a statement above; however, since C# 7, `throw` has been usable as an expression.  This makes it possible to throw expressions a bit more succinctly in some cases.  For example, the situation where a constructor parameter needs to be assigned to a field, and an exception thrown if the parameter was null, can now be written idiomatically in a single line of code using the null-coalescing operator and a `throw` expression.  Code that would have previously looked like this:

```
public Oven(Fuel fuel)
{
    if (fuel == null)
    {
        throw new ArgumentNullException("fuel");
    }
    _fuel = fuel;
}
```

can now be written as:

```
public Oven(Fuel fuel)
{
    _fuel = fuel ?? throw new ArgumentNullException(nameof(fuel));
}
```

The null-coalescing operator `??` will be explained more fully below.

Exceptions are trapped, to prevent them from causing the program to exit, by putting your code within a `try` block, and adding `catch` blocks to determine which exceptions to handle.  The simplest case is a catch block which will handle almost any exception, bar a few edge cases.

```
try
{
    // ... code goes here
}
catch
{
    // if an exception is thrown in the block above, execution will transfer to here.
}
```

If you declare the exception object, you can refer to it within the catch block:

```
try
{
    // ... code goes here
}
catch (Exception e)
{
    Console.WriteLine(e.Message);
}
```

The above catches, which will catch any exception (again, barring strange or extreme edge cases), are sometimes known as a "pokemon catch" from their ability to "catch them all".  They are considered bad practice to some extent, but are useful in occasional circumstances: for example, if you want to ensure that a service will continue running regardless of almost any sort of error.

A catch block can throw a new exception, or can rethrow the existing exception.  The former case is the same as a `throw` anywhere else; the latter case is done with a `throw` statement with no exception given:

```
try
{
    // ... code goes here
}
catch (Exception e)
{
    Console.WriteLine(e.Message);
    throw;
}
```

Any code after the `throw;` would be unreachable.  Sometimes you will see code where this is written as `throw e;`&mdash;this works, but is generally considered a bad idea as it will re-set the `StackTrace` property of the exception as if you were throwing a new instance, whereas the version above does not.  If you do throw a new exception instance, either of the same type or a different type, the correct thing to do is to set the `InnerException` property of the new exception to the original exception, like so:

```
try
{
    // ... code goes here
}
catch (Exception e)
{
    Console.WriteLine(e.Message);
    throw new SpecialistException("Error occurred in business logic processing) { InnerException = e };
}
```

That way, the stack trace showing the exact location the exception was originally thrown is preserved for later analysis.

Catch blocks can be more restrictive, and only catch exceptions which are of a certain class or derived classes.  In C# 6 and onwards, they can also be filtered using a `when` clause, which takes any boolean expression.  If an exception could match multiple catch blocks, the first one to match is called.  Only one catch block is called; there is no fallthrough from one to the next.

```
try
{
    // ... code goes here.
}
catch (ArgumentOutOfRangeException e)
{
    // handles all ArgumentOutOfRangeException instances, or any of its descendants.
}
catch (ArgumentException e)
{
    // handles all ArgumentException instances, or any of its descendants - except ArgumentOutOfRangeException, because that was handled above.
}
catch (Exception e) when (e.Message.Contains("biscuits"))
{
    handles all exceptions that are not an ArgumentException (or descend from it), that mention biscuits in their error message.
}
```

You can also define a `finally` block.  This goes after the final `catch` block, and contains code that is (almost) always run.  If the `try` block executed successfully with no exceptions, the `finally` block is then executed before continuing.  If an exception is thrown and handled by a catch block, the `finally` block is run after the catch block executes; if the catch block throws or rethrows an exception, the `finally` block is run before the new exception is handled.  The `finally` block is intended for cleanup code that must normally always be run, for example to close open files or network connections.  You can, indeed, use `try { ... } finally { ... }` with no `catch` for this purpose.

I said the code in a `finally` block is *almost* always run.  If an exception bubbles up all the way to the top of the stack and the program exits as a result, it might not run; whether it runs or not depends on various system-specific conditions such as what CLR implementation is running the code and how the operating system is configured.  Normally this is not a problem given that your program is in its death-throes anyway; if you absolutely must guarantee that a `finally` block is run then insert a bare-bones `catch (Exception e) { throw; }` block either above the `finally` in question, or if that is not possible, in a calling frame.

You may see code in which exception handling is used for "normal" flow-of-control&mdash;code in which exception handling is used not just to trap error states, but also to detect and trap normal, expected situations and behave accordingly.  In some situations this can be very convenient to do; however, it is generally not recommended.  There is something of a performance penalty imposed when using try and catch, due to the amount of work which may be involved in unwinding the stack and generating the stack trace, so using them for normal flow-of-control should be avoided if possible.

#### Expressions and operators

Expressions are such a key part of the language that it feels slightly strange to be approaching them at this point in the course, after we have already covered more complex topics such as classes, inheritance, exception handling and suchlike.  After all, the vast majority of statements in C# are either expressions or contain them, to such an extent that you are liable to overlook their existence, unable to see the wood for the trees.  In essence, anything that is not either a declaration of some kind, or one of the kinds of statement listed above, is probably an expression, and many of those statements themselves contain expressions, as we have seen.

Expressions can contain operators; indeed, expressions are the only place you will see operators.  However, an expression does not have to contain any operators; for example, a variable name on its own can be an expression, such as when it is the parameter to a method call.  A literal value is also itself an expression, as is a method call.

Every expression has a type, and when evaluated, a value.  The type of an expression is the type of the value it evaluates to, but not necessarily the specific type it will evaluate to.  This is self-evidently the case if an expression's type is one that cannot be instantiated, such as an interface or an abstract class.

The language has a relatively rich set of operators, which can be categorised in a variety of ways.  There are unary, binary and ternery operators, taking one, two or three operands respectively; there are operators with side-effects and operators without side-effects.  Most C# operators are *left-associative*; that is, they group from left to right by default, but a few are right-associative.  The operator order of precedence can be overridden using parentheses in an intuitive way.  There are a number of situations where the same symbol is used for both a binary and an unary operator.

All operands are themselves expressions, and are evaluated from left to right irrespective of the associativity of the operator.  Some operators place restrictions on their operands; for example, the left-hand side of an assignment expression must be something that can have a value assigned to it, such as a mutable variable or a settable property (similar to the concept of an *l-value* in C, although in C# the concept does not have a name).  The majority of binary operators always evaluate both operands; some binary operators and the "conditional operator" (the only ternery operator) conditionally evaluate some operands.

When operators are used with numeric operands, a set of promotion rules is applied to the types before the operation, and this determines the type of the result.  Notably, the smallest-sized result of any numeric binary operator is `int`, so even, say, a bitwise operation between two bytes will always give an `int` value as the result.  In general automatic promotion always results in a type which can contain any possible value of either operand, with an exception thrown if this is not possible: for example, an operation between an `ulong` and and `int` without explicit casting will throw an exception, because there is no type which can store both the largest values of `ulong` and the negative values of `int`.  Similarly, you cannot mix `decimal` with either `double` or `float` operands without using explicit casting.

Some operators can be *overloaded* and have their behaviour redefined for user-defined types.  The syntax for this is described below.

Some expressions have their values fixed at compile time.  Literal expressions are one example, as are operator expressions whose operands are all literals; the value of `nameof` expressions, and some values of `sizeof` and `default` expressions.

The full operator precedence table is below.  Operators in the same row take precedence according to their associativity.  The operators are explained more fully after the table.

| Operators | Classification | Notes |
| --- | --- | --- |
| `obj.Member`, `obj?.Member`, `obj[i]`, `obj?[i]`, `Method()`, `x++`, `x--`, `new`, `typeof()`, `default`, `nameof()`, `delegate`, `sizeof()`, `stackalloc`, `p->Member`, `checked`, `unchecked` | Primary operators | `obj?.Member` and `obj?[i]` are conditional.  `p->Member` is discussed in the section below on unsafe code.  `delegate` is discussed in the section on lambdas and delegates. |
| `+x`. `-x`, `!x`, `~x`, `++x`, `--x`, `(T)x`, `await`, `&x`, `*x`, `^x`, `true`, `false` | Most unary operators, other than those in the row above | `await` is discussed in the section below on asynchronous code.  `*x` is discussed in the section on unsafe code. |
| `x..y` | The range operator | |
| `switch`, `with` | | `with` is discussed in the section on record types. |
| `x * y`, `x / y`, `x % y` | Multiplication, division and modulus | |
| `x + y`, `x - y` | Addition and subtraction | |
| `x << y`, `x >> y` | Bit-shifting operators | |
| `x < y`, `x <= y`, `x > y`, `x >= y`, `is`, `as` | Relational and type-testing operators. | Note that "greater than or equal to" is written `>=`, not `=>` |
| `x == y`, `x != y` | Equality and inequality | |
| `x & y` | Boolean AND | Either bitwise or logical depending on the type of the operands |
| `x ^ y` | Boolean XOR | Either bitwise or logical depending on the type of the operands |
| `x \| y` | Boolean OR | Either bitwise or logical depending on the type of the operands |
| `x && y` | Logical conditional AND | |
| `x \|\| y` | Logical conditional OR | |
| `x ?? y` | Null-coalescing operator | Right-associative and conditional |
| `c ? t : f` | The conditional operator | Right-associative and conditional |
| `x = y`, `x += y`, `x -= y`, `x *= y`, `x /= y`,  `x %= y`, `x &= y`, `x \|= y`, `x ^= y`, `x <<= y`, `x >>= y`, `x ??= y`, `=>` | Assignment operators and the lambda definition operator | The lambda definition operator (`=>`) is discussed in the section on lambdas and delegates. |

As you can see in the table, most binary operators are normally written with a space either side of the operator symbol(s), but some, such as the member access operator, are not.  Unary operators are normally written with no space between the operator and the operand, and almost always precede the operand, the exceptions being `x++` and `x--`.

The member access and indexer access operators (`obj.Member` and `obj[i]`) have already been seen above.  The variants `obj?.Member` (occasionally called the "Elvis operator" due to its appearance) and `obj?[i]`, introduced in C# 6, are the null-conditional access operators.  If the left-hand operand is not null, they behave identically to the normal member access and indexer access operator.  If the left-hand operand does evaluate to null, the right-hand side of the operation is not evaluated and the value of the expression is null.  This enables you to write chains such as `a?.B?.C?.D` which will successfully return null if any of `a`, `a.B`, `a.B.C` or `a.B.C.D` are null.  With the normal member access operator, only the last of these situations would successfully return null; the others would throw a `System.NullReferenceException`.  The null-conditional operators were introduced in C# 6, and ever since have prompted much debate as to whether they are a convenient addition to the language, or promote lazy coding.

The `nameof()` operator was briefly discussed above.  Its value is computed at compile-time: it is a string constant which is the name of the symbol passed to it.  The `typeof()` operator, although named similarly, is computed at runtime.  Its operand is a type name identifier, and it returns a `System.Type` instance containing that type's metadata.  For example: `Type stringType = typeof(string);`.

The `switch` operator and its syntax are covered above alongside switch statements, under "Flow of control".  Its left operand is an expression of any type, and its right operand is a list of patterns and expressions; the value of the switch expression is the value of the expression whose pattern best matches the value of the left operand.  The other "expression arms" are not evaluated.

The operators `x++` and `x--` are the post-increment and post-decrement operators.  As in other languages, their value is the value of `x`, but they have the side-effect of increasing or decreasing `x`'s value by one.  By contrast the operators `++x` and `--x` are the pre-increment and pre-decrement operators.  They increase or decrease the value of `x` by one, and the value of the expression is the value of `x` after the increment or decrement.

The range operator `..` was introduced in C# 8.  Its operands must be of either `int` or `System.Index` type (or implicitly convertible to them), and its result is of type `System.Range`.  Its left operand is the inclusive lower bound of a range of indices, and the right operand is the exclusive upper bound.  Either operand may be omitted to give a range with no lower or no upper bound.

The unary `^x` operator is the "index from end" operator, and was introduced alongside the `..` operator.  Its operand is an `int`, and its result is of type `System.Index`.  For a sequence of items of length `x`, `^x` means the index `length - x`.  In other words, the last item in a sequence can be indexed as `^1`.

The unary `^` is often used with the `..` operator.  For example `..^1` means "all elements aside from the last", and `^3..` means "the last three elements.

The `default` operator gives the default value of a type.  Historically it had to be used in the same way as `typeof()`, with the type name in parentheses such as `default(string)`, but since C# 7.1 it can be used on its own in contexts where the compiler can infer the type required.

The unary `+x` operator essentially has no effect, as its value is the value of its operand.  `!x` is the logical NOT operator and `~x` is the bitwise NOT operator.

The operator `(T)x` is the casting operator, used to forcibly convert one type to another.  If this is not possible, it will throw an exception at runtime.

The `true` and `false` operators are rarely used, but can be implemented by a type to specify a conversion operation, to indicate that the operand is either definitely true or definitely false.  Note that they are not required to be defined as complements.

The `>>` right-shift operator's behaviour is type-dependent.  On signed types, it performs an arithmetic shift and populates the high-order bit(s) with the value of the most significant bit.  On unsigned types, it populates the high-order bit(s) with zero.

The `is` operator was originally intended for type-testing.  With expressions of the form `expr is type`, where `expr` is an expression and `type` is a type name, the operator's value is true if the expression's type is `type` or is derived from `type`&mdash; in other words, if the operand could be assigned to a variable of type `type` without error.  If not, its value is false.  Since C# 7 the `is` operator has slightly extended syntax, the most useful variant of which is a format that, if the operator is true, assigns the expression to a newly-declared variable of the type.  This syntax reads, for example, `expr is int x`: if `expr` can be assigned to the `int` type, it will be assigned to a newly-declared `int` variable `x`.  The C# 8 and later version of `is` also allows you to use a constant as the right-hand operand; this is exactly equivalent to using the `Equals()` method, but allows you to write `expr is null` if you want to write that&mdash;it is a useful way of avoiding infinite loops when defining overrides of the `==` operator.  It also introduces the syntax `expr is var x`, which is almost equivalent to writing `var x = expr` but is an expression, rather than a declaration statement.  C# 9 extended this to include relational and boolean-logic patterns, so you can write `expr is >= 0 and < 256`.

The `as` operator allows you to do safe type-casting.  If it is possible to convert an expression `expr` to type `T` then the expression `expr as T` is effectively the same as `(T)expr`.  However, if it is *not* possible, the former will be `null` without error whereas the latter will throw a `System.InvalidCastException`.  The `as` operator gets less use in new code due to the C# 7+ extensions to the `is` operator; code that would have formerly been written:

```
var castValue = expr as T;
if (castValue != null)
{
    // code here...
}
```

can now be written slightly more succinctly as:

```
if (expr is T castValue)
{
    // code here...
}
```

The Boolean operators are `&`, `&&`, `|`, `||` and `^`.  The single-symbol versions are unconditional&mdash;they always evaluate both operands&mdash;and are either bitwise or logical depending on operand type: the former for numeric integer types, the latter for `bool`.  Note that as described above the smallest type that can result from a bitwise operation is always `int`, even if the operands are `byte`, so low-level bit-manipulation code which operates on individual bytes can potentially involve a lot of casting back to `byte` unless the values can be packed into `int`s first.  Note also that the logical `^` operator always gives the same result as `!=`, but has lower precedence.

The two-symbol Boolean logical operators `&&` and `||` are conditional operators, also known as "short-circuiting" operators.  They always evaluate their left-hand operand, but only evaluate their right-hand operand if it will make a difference to the final value of the expression.  In other words, `&&` only evaluates the right-hand operand if the left-hand operand is true, and `||` only evaluates the right-hand operand if the left-hand operand is false.  There is no such operator as `^^`, because its behaviour would be no different to `^`.

The `??` operator is the null-coalescing operator, introduced in C# 6, and is also conditional.  If the left-hand operand evaluates to a non-null value, that is the value of the expression and the right-hand operand is not evaluated.  If the left-hand operand is null, the right-hand operand is evaluated and becomes the value of the expression.  This is one of the few operators with right-associativity, so that `x ?? y ?? z` is equivalent to `x ?? (y ?? z)`, not `(x ?? y) ?? z`.

The `? :` operator is the only ternery C# operator, and is also conditional and right-associative.  Given an expression of the form `test ? x : y`, the `test` expression is first evaluated.  If it is true, `x` is evaluated and becomes the value of the expression; if false, `y` is evaluated and becomes the value of the expression.

Aside from the lambda-definition operator `=>` ("fat arrow"), which is described in a later section, all of the operators in the lowest-precedence rung of the table are assignment operators of one type or another.  Aside from the straightforward assignment operator `=`, each is an assignment form of a binary operator which also assigns the value of the expression to the left-hand operand.  For example, to take the best-known operator in this family, `x += y` is equivalent to `x = x + y` aside from the fact that `x` is only evaluated once.  There is a rich set of other assignment operators in the family, though; the `??=` operator, the newest, was introduced in C# 8.  `x ??= y` sets `x` to `y` if `x` was previously `null`.

Most operators can be overloaded to handle user-defined types where appropriate.  This is done by defining a static, public "operator method" within the operand type (or one of the operand types, if they differ).  The name of the method is the symbol of the operator, and the declaration includes the `operator` keyword after the return type.  For example, if you wanted to define the equality operator for an `Egg` class it might look something like this:

```
public static bool operator ==(Egg a, Egg b)
{
    if (ReferenceEquals(a, b))
    {
        return true;
    }
    if (a is null || b is null)
    {
        return false;
    }
    return a.MajorAxisLength == b.MajorAxisLength && a.Circumference == b.Circumference && a.Material == b.Material;
}
```

The above method starts by using the `object.ReferenceEquals()` method to determine if either the two parameters refer to the same object, or both are null.  The second `if` will return false if one of the parameters is null (we could also use the `^` or `!=` operators, but `||` is slightly clearer in its intent), and finally we return true if all of the objects' properties are equal.

This example avoids a trap that is easy to fall into the first time you write an operator method.  You might start out with something along these lines:

```
public static bool operator ==(Egg a, Egg b)
{
    if (ReferenceEquals(a, b))
    {
        return true;
    }
    if (a == null || b == null)
    {
        return false;
    }
    return a.MajorAxisLength == b.MajorAxisLength && a.Circumference == b.Circumference && a.Material == b.Material;
}
```

If you do, though, you've immediately fallen into a tight infinite loop.  The `a == null` expression will be turned into a recursive call into your operator method, and after a few seconds your code will crash with a `StackOverflowException`.  Using `a is null` gets around this in C# 7 or later; the older way to avoid this trap is to use `ReferenceEquals(a, null)` instead.

Some operators are restricted in that they cannot be defined alone.  The `==` operator is an example of this: if it is defined for a particular pair of operand types, then `!=` must also be defined for the same pair.  Similarly, you can't define `>` without also defining `<`.  You are allowed to define `>` without `>=`, but in most cases it probably makes sense to do so.  You should also consider when you need to implement operator commutativity yourself, where the operator is one that people would intuitively expect to be commutative; commutativity is never assumed by the compiler.  In other words, if you define `operator ==(TypeA a, TypeB b)` you should also implement `operator ==(TypeB b, TypeA a) => a == b;` to ensure that the `==` applied to that pair of types is guaranteed to be commutative with only one significant implementation to maintain.  Whether both operators should be implemented together, or one in `TypeA` and the other in `TypeB` is a matter for debate.

If you implement the equality operator for a particular class, you should also by convention implement the other way to test for equality, the `Equals()` method (and vice versa).  If your type `Egg` implements `operator ==(Egg a, Egg b)` it should also implement the `IEquatable<Egg>` interface (which contains the `Equals()`) method and override the `object.Equals()` and `object.GetHashCode()` method.  This latter method should return an `int`.  It must return the same value for any two instances which the equality operator would describe as equal, and should usually return different values for any two instances which the equality operator would describe as unequal; this can include the same instance at different points in time.  If the `GetHashCode()` method does not behave as expected, some library features may perform poorly when used with the given types.

#### Interfaces

Interfaces have already been mentioned above, briefly, but without delving into their details.  As we outlined earlier, an interface is a reference type, but all its members are implicitly abstract and virtual.  In many ways it is like an abstract class, but with the primary difference that interfaces cannot contain anything which preserves per-instance state.

* Interfaces cannot contain instance fields.
* Interfaces cannot contain auto-implemented properties, although they appear to.  Interface properties that appear to be auto-implemented properties are implicitly abstract.
* As classes can only inherit from a single superclass, concrete implementations of an abstract class cannot inherit from any other classes.  However, classes can implement any number of interfaces.
* Structs cannot inherit from other structs.  However, they can implement any number of interfaces.
* Interfaces cannot be derived from classes, but can be derived from any number of interfaces.

Before C# 8, interfaces could not contain concrete implementations, constants, operators, nested types, static members, and explicit access level modifiers.

Some of the above will change from C# 8 onwards: in that version of the language, interfaces will be permitted to provide default implementations of members, making them more like abstract classes allowing multiple inheritance.

In versions of C# before version 8, a non-abstract class that implements an interface must implement every member of the interface, and the implementations must be marked as `public`, otherwise you will receive a compile-time error.  The names and types of the members must match; when implementing methods, the types of all the parameters must match.  The names of the parameters do not have to match, but it is good practice to ensure that they do.

If multiple interfaces declare an identical member, then normally if a class implements all of them a common implementation suffices.  In other words, if you have two interfaces:

```
public interface IEgg
{
    void Grow();
}
public interface IZygote
{
    void Grow();
}
```

then a class only needs to provide a single `Grow()` method to implement both.

```
public class Egg : IEgg, IZygote
{
    public void Grow()
    {
        // code goes here...
    }
}
```

However, if you need to define different behaviour for each interface, then you can use "explicit interface implementation" to specify that an implementation belongs to one particular interface.  Which method gets called will then depend on the type of reference used to access it.  An explicit interface implementation is written by prefixing the member name with the interface name, separated from it by a dot.  Here is an example:

```
public class Egg : IEgg, IZygote
{
    public void Grow()
    {
        Console.WriteLine("Egg.Grow() called");
    }
    public void IZygote.Grow()
    {
        Console.WriteLine("IZygote.Grow() called");
    }
}
```

With the above class, then the following code:

```
Egg egg = new Egg();
egg.Grow();
(egg as IEgg).Grow();
(egg as IZygote).Grow();
```

would produce the following output:

```
Egg.Grow() called
Egg.Grow() called
IZygote.Grow() called
```

Note that the explicit implementation `IZygote.Grow()` has been called, but as an explicit implementation was not provided for `IEgg.Grow()`, the `Egg.Grow()` implementation was used instead.

You must also use explicit interface implementation syntax when writing an interface which overrides a method defined in a base interface.

#### Attributes

Attributes are a way for the developer to attach custom metadata to code: to assemblies, classes, members, method parameters and method return types.  Some standard attributes affect how the compiler behaves; others are provided by the system or by frameworks to control runtime behaviour, and developers are also free to define their own custom attributes.  A few examples of the things attributes can be used for are:

* Controlling how C# code interacts with native code.
* Marking code as obsolete.
* Controlling serialisation frameworks.
* In test projects, marking which methods are to be run as tests.
* Providing metadata to dependency injection frameworks.
* In web API or MVC projects, defining how URIs map to code methods, or linking up custom authenthication or error routines.

Attributes are written in square brackets, and precede the declaration they apply to.  For example, to mark a method as obsolete you add the `[Obsolete]` attribute as follows:

```
[Obsolete]
public void VersionOneMethod()
{
    // ...
}
```

Attaching an attribute to a declaration is known as "decoration".  When the compiler sees code using something that has been decorated with the `[Obsolete]` attribute, it will emit a warning.  A given declaration can be decorated with any number of attributes.

Each attribute is a class which must be derived in some way from the `System.Attribute` class.  By convention the class name of an attribute ends in `Attribute`, and if it does, you do not need to include the `Attribute` suffix when it is used.  For example, `[Obsolete]` is implemented by the class `System.ObsoleteAttribute`.  It's not necessary to follow the convention; if you do decide to go against it, you use the whole class name.

Attributes can have parameters, which can be compulsary or optional, and can be positional or named.  For example, the `[Obsolete]` attribute has an optional positional parameter which customises the warning message emitted by the compiler:

```
[Obsolete("Use VersionTwoMethod() instead")]
public void VersionOneMethod()
{
    // ...
}
```

The rules on the usage of positional and named parameters together is similar to methods: positional parameters must come before any named parameters, but other than that named parameters can be given in any order.  The syntax for named parameters is slightly different to methods, though: where methods use `:` to separate name and value, attributes use `=`, as per the following example.

```
[DllImport("user32.dll", SetLastError=false)]
```

The values of attribute parameters have similar restrictions to the values of `const` field initialisers: they must be a compile-time constant, which effectively limits them to being either a literal, or the result of a compile-time operator expression such as `nameof`, or a `typeof` operator (normally a runtime operation but allowed to be compile-time for this specific case).  Some Microsoft documentation states they cannot be an expression; strictly speaking this is wrong, because literals are themselves expressions, but they must be hardcodable at compile time.  One example of a commonly-used attribute normally used with a compile-time operator is in the Visual Studio unit testing framework, where `ExpectedExceptionAttribute` is used to assert that a test must throw a specific exception in order to pass.  A test to assert that the `List<T>` class throws an exception if you try to remove an element that does not exist might be written as follows:

```
[TestMethod]
[ExpectedException(typeof(ArgumentOutOfRangeException))]
public void ListOfIntClass_RemoveAtMethod_ThrowsExceptionWhenListIsEmptyAndParameterIsZero()
{
    List<int> testObject = new List<int>();

    testObject.RemoveAt(0);
}
```

Note the two attributes, firstly to mark this method as a test, and secondly to indicate the exception type that the test will throw.  Note also that this test has no explicit assertion code&mdash;assertion is handled in this case by the framework.  The class containing this method will need to be decorated with the `[TestClass]` attribute in order for the Visual Studio test runner to pick up the test.

Custom attributes can be created by creating a class that derives from `System.Attribute` or any of its descendents.  The parameters to the attribute class's public constructor(s) are the attribute's positional parameters, and the class's public-settable properties (or public fields) are its named parameters.  You can limit the targets of an attribute class (for example, to say that an attribute can only be used to decorate a method, not a class) by decorating the attribute class with the `[AttributeUsage]` attribute (for example, `[AttributeUsage(AttributeTargets.Method)]`).

If the target of an attribute is potentially ambiguous, it can be specified by preceding the attribute name with the kind of target and a colon.

Attribute classes are not instantiated until the metadata of the relevant item is inspected through reflection.  Given an object reference `x`, the code `object[] attribs = x.GetType().GetTypeInfo().GetCustomAttributes(false);` will return an array of instances of the attributes that decorate the specific type of `x`, and changing the `false` parameter to `true` will return all attributes that decorate all of the types `x` is derived from.  Note that a second call to `GetCustomAttributes()`, even on the same `TypeInfo` object, will return new attribute instances.  For attributes on other kinds of declaration, in most cases there is a `GetCustomAttributes()` method which can be called on a `MemberInfo`-derived class appropriate to the kind of declaration; the most significant exception to this is attributes applied to the return type of a method, which are accessed through the `MethodInfo.ReturnTypeCustomAttributes` property.

#### Literals

Like expressions, we have already had literals appear many times in the examples, so it might seem a bit strange to include them at this point!  There are a few bits, though, which haven't been touched upon so far that are fairly straightforwards and that are worth mentioning.

Numeric literals are by default `int` if they are decimal, hexadecimal or (since C# 7) binary integers small enough to fit into an int, and `double` if they include a decimal point or are in exponential notation.  Since C# 7, integer literals can include underscores, which are ignored, for clarity.  The hexadecimal and binary integer literal formats are indicated with case-insensitive `0x` and `0b` prefixes.

```
var a = 42;            // int
var b = 0x2a;          // int
var c = 0B001010101;   // int
var d = 42.0;          // double
var e = 4.2e1;         // double

var f = 37.62372;      // double
var g = 1e6;           // double
var h = 1_000_000;     // int, with the same value as g, written with underscores for clarity.
```

If an integer literal is too large to fit into an `int`, its type will be the smallest type it fits into out of `uint`, `long` and `ulong`, with `long` preferred over `ulong`.  Integer literals that fit into an `int` can be assigned to a `nint`, and those that fit into a `uint` can be assigned to a `nuint`, but the type of the variable has to be explicitly declared.

```
// These examples are all arbitrary numbers, not edge cases!
var i = 73;                        // int
var j = 3_296_874_906;             // uint
var k = -3_296_874_906;            // long
var m = 9_294_102_466_702;         // long
var n = 9_372_372_278_864_032_911; // ulong
```

As mentioned earlier, each type does have a `MaxValue` member to give you the definitive maximum limit should you need it in code.  FOr most types (except `nint` and `nuint`) these are `const` fields, so can be used as compile-time constants; for `nint` and `nuint` they are properties computed at runtime as they may vary between systems.

You can override the default type of a numeric literal, if legal, using a suffix.  The `L` suffix specifies `long` and `UL` specifies `UL`.  However, a number with an `L` suffix that is too large for a `long` will be promoted to `ulong` without error.

```
var a = 42L;                        // long
var b = 42UL;                       // ulong
var c = 9_372_372_278_864_032_911L; // ulong, despite the explicit `long` suffix.
```

The suffixes are case-insensitive, so if you wanted you could write `var a = 42l;`.  Please don't.

The equivalent suffixes for real types are `d` for `double`, `f` for `float` and `m` for `decimal`.  They are also case-insensitive, and the majuscule forms are a little less problematic.

```
var a = 42d;     // double
var b = 3.8f;    // float
var c = 1e-2m;   // decimal
```

The `d` suffix is only really relevant for integer literals, because unsuffixed literals with a floating point default to `double`.

Although there are no suffixes for the smaller integral types, assignment of integer literals to them will still work if the type is explicitly defined and the constant is within the correct range.  You can put an explicit cast in if you feel it adds clarity or if it is necessary.

```
byte a = 42;           // legal even though 42 defaults to being an int constant
DoSomething((byte)42); // A hypothetical method with DoSomething(int x) and DoSomething(byte x) overloads; we have explicitly cast the constant to get the overload we want.
```

String literals come in three flavours: regular, verbatim and interpolated.  Regular string literals are the sort most often seen, delimited by `"`.  Within a regular string literal, the backslash character is used to define an escape sequence, used to insert a delimiter or a control character.  The C# escape sequences are as follows, with `H` representing a compulsary and `h` representing an optional hexadecimal digit:

| Escape sequence | Meaning |
| --- | --- |
| `\'` | Single quote |
| `\"` | Double quote |
| `\\` | Backslash |
| `\0` | NUL character (ASCII 0) |
| `\a` | Alert (Ctrl-G, ASCII 7) |
| `\b` | Backspace (ASCII 8) |
| `\f` | Form feed (ASCII 12) |
| `\n` | Newline |
| `\r` | Carriage return |
| `\t` | Horizontal tab (ASCII 9) |
| `\v` | Vertical tab (ASCII 11) |
| `\uHHHH` | UTF-16 escape sequence |
| `\U00HHHHHH` | UTF-32 escape sequence |
| `\xhhhH` | Variable-length UTF-16 escape sequence |

The `\u` and `\x` sequences differ only in that with the latter leading zeros may be omitted.  If the following character in the string is a valid hex digit which would inadvertently extend an `\x` sequence, the `\u` and `\U` sequences can be used instead.

Line breaks are not permitted inside regular string literals.

Verbatim string literals are prefixed with an `@` character.  In a verbatim string literal, there are no backslash escape sequences, and line breaks are permitted.  The only escape sequence within a verbatim string literal is `""`, which indicates one `"` character.  At one time verbatim string literals were much used to write Windows file paths, as they avoided doubling up all the directory-separator backslashes: it is much clearer to write `@"C:\Users\Caitlin\Desktop\Stuff\Repos\Guides\c-sharp-crash-course.md"` than to write `"C:\\Users\\Caitlin\\Desktop\\Stuff\\Repos\\Guides\\c-sharp-crash-course.md"`.  Nowadays, of course, in the platform-agnostic world you probably shouldn't be hardcoding paths into your code in that way.  Verbatim string literals are converted into regular string literals at compile time, so if you decompile your code or inspect it in the debugger, all the escape sequences will be visible.

Interpolated strings were introduced in C# 6, and work a bit like strings in Perl or similar languages: they enable you to embed expressions into your string literal that will be interpreted at runtime.  They are prefixed with the `$` character, and the embedded expressions within the string are surrounded with braces, like this:

```
$"There are {lines.Count} lines in the document."
```

In this example, `lines.Count` is an expression that will be evaluated, converted to a string, and inserted into the literal at runtime.  Generally speaking, interpolated strings are converted at compile time into calls to the `string.Format()` method, which is discussed more fully below.  From C# 10 onwards, you can also use interpolated strings in `const` initialisers as long as all the expressions in the string are constant strings; you can't use any expressions whose string conversion might vary at runtime, including numeric compile-time constants.

The backslash escape sequences from regular string literals also work in interpolated strings.  C# 8 introduced verbatim interpolated strings, enabling constructs such as `$@"C:\Users\{user}\Desktop"`, which is a bad example you should never actually use in the real world.

#### Indexers

Hopefully you're already familiar with the idea of accessing members of an array through square brackets: `int x = arr[i];` for example.  In C this syntax is just a synonym for pointer arithmetic.  In C#, however, the square bracket operator can be extended to apply to any class for which it makes sense.  This is done by defining one or more *indexers*, effectively special properties which take one or more parameters to determine which element of the class's members you would like to access.  In code, an indexer is defined as if it were a property named `this[...]` with its parameters inside the square brackets.

For an example, we may as well use the source code of .NET Core itself.  This was (when I originally started writing this) the main-branch code for the indexer of the `System.Collections.Generic.List<T>` class, probably the most widely-used array-like class in C#.  The `List<T>` class's backing store is an array of type `T[]` called `_items`, so its indexer essentially just proxies that array, after doing range checks.  The `_version` field is a private field used by other methods to detect if the contents of the list have changed.  The below code was copied directly from GitHub, including the comment.

```
public T this[int index]
{
    get
    {
        // Following trick can reduce the range check by one
        if ((uint)index >= (uint)_size)
        {
            ThrowHelper.ThrowArgumentOutOfRange_IndexException();
        }
        return _items[index];
    }

    set
    {
        if ((uint)index >= (uint)_size)
        {
            ThrowHelper.ThrowArgumentOutOfRange_IndexException();
        }
        _items[index] = value;
        _version++;
    }
}
```

As you can see above, the code for an indexer is essentially a property with one or more parameters; like a property, the `set` method also takes a `value` parameter.  Indexers can have multiple parameters (like a multi-dimensional array), and the parameter(s) can be of any type.  The most common parameter types are integers (for array-like data structures) or strings (for hash-table-like structures).  Traditionally other parameter types are equally legal but generally discouraged; since C# 8, if your type has an `int` indexer you will probably also want to provide `System.Index` and `System.Range` indexers so that calling code can use the `..` and unary `^` operators fully.

Like properties, indexers can be defined in interfaces, and can be read-only, write-only and read-write.  Like properties, since C# 6 read-only indexers can be expression-bodied (defined using `=>`) and since C# 7.0 any indexer can be expression-bodied, using the same syntax as for properties.  Unlike properties, indexers cannot be `static`; because of this an indexer defined in an interface as `{ get; }` or `{ get; set; }` will need to be implmented in any implementing classes or structs like a property would have to be.

Like methods, indexers can be overloaded with different parameter type signatures.  For example, the `System.Data.IDataReader` interface, which is used to define classes for carrying out forward-only reading of rows of tabular data, contains two indexers, one which takes an `int` parameter to access data fields by column index, and one which takes a `string` parameter to access data fields by column name.  The same overloading rules apply as for methods: a type cannot define two indexers with the same parameter signature and different types.

### Structs

Structs have been discussed briefly above: they are in many ways like classes, but they are value-typed.  Under the hood, all of the built-in numeric types are structs, so in many ways their behaviour is fairly intuitive, if you think of it in terms of "how would a numeric variable behave here?"

When a struct is passed in as a normal method parameter, as a value type it is copied, which could impose a large penalty for a large struct.  This is a shallow copy, so all values are copied as-is and any reference-type members will point to the same objects as before.

All structs are implicitly `sealed` so cannot be inherited.  Because of this, access levels that are meaningless for members of `sealed` types, such as `protected` or `virtual`, are not permitted on struct members, and they and their members cannot be declared `abstract`.  The base type of all structs is `ValueType`, and a base type cannot be defined in code.

Structs can implement interfaces, and can contain the same kinds of members as a sealed class.  All structs must have a parameterless constructor.  This can be the implicit parameterless constructor inserted for a type that has no defined constructors; but if a struct has any defined constructors, it must have a parameterless constructor and this constructor must set the values of all instance fields and properties.  For structs, the parameterless constructor is called by the `default` operator, whereas for classes the `default` operator returns `null`.

Structs cannot be declared `static`, but they can have `static` members.

Normal C# coding standards recommend that a struct of type `S` should implement `IEquatable<S>`, and should override `object.Equals()`, `object.GetHashCode()`, and the `==` and `!=` operators.  This is not compulsary, but it avoids boxing issues which would otherwise result in `object.Equals()` and `==` always returning `false` for the type, whatever the values being compared.

### Records

Record types were introduced in C# 9, with further changes in C# 10.  They are divided into record classes (reference types) and record structs (value types).  Code or documentation that refers to "records" alone may be referring to record classes, or may be referring to the things that record classes and record structs have in common.

#### Record classes

Record classes (sometimes just called records) were introduced in C# 9 and were intended as a way for developers to easily create immutable data record types.  Under the hood, if you decompile your CIL, you will see that record classes compile to classes with a lot of autogenerated boilerplate; but within the syntax of the language they are a distinct kind of type.  The simplest type of record definition uses *positional parameters* and consists of a single line of code:

```
public record Vehicle(string Registration, string Colour);
```

This will create a record class type called `Vehicle`, with the following:
- A constructor whose signature matches the declaration.
- Immutable (get-init) properties whose names and types match the parameters, and whose access levels match their containing record class.
- A second constructor that takes a `Vehicle` parameter, which acts as a copy constructor.
- An anonymous method which uses the copy constructor to clone an instance of the type.
- A `Deconstruct` method with `out` parameters matching the constructor parameters: in our example this would have the signature `public void Deconstruct(out string Registration, out string Colour)`.  This method can be implicitly called using deconstruction syntax (see below).
- A property named `EqualityContract` which returns the type of the instance.
- An `Equals(Vehicle other)` method, so the type implements `IEquatable<Vehicle>`.
- An override of `object.Equals(object other)`.
- Overrides of the `==` and `!=` operators.
- An override of `object.ToString()` which outputs the values of each property.  The output of this for our example `Vehicle` type would be `Vehicle { Registration = FAB 1, Colour = Pink }`.
- A method named `PrintMembers()`, which is used by the `ToString()` method to format the properties of the record and their values.  This method will be either `protected` or `private` depending on the modifiers of its containing type.

Positional record declarations are the only place in the normal C# coding standards, incidentally, where it is normal to define a parameter name that begins with a capital letter, so that the generated property names follow the normal standard.  From C# 10 onwards you can declare a record class with the keywords `record class`, which are a synonym for `record` alone.  Note that if you use a positional parameter record class declaration, the record class will not have a parameterless constructor created.

The `Equals()` methods and operators implement equality by checking the values of each property.  This is a shallow check, unless the properties are custom types with deeper equality checking.

If you need to add attributes to the properties of a record class declared with positional parameter syntax, you can put attributes on the parameters but add the `property` target to the attribute, as shown here:

```
public record Vehicle([property: JsonPropertyName("vrn")] string Registration, string Colour);
```

You can add your own members to a record class just like a regular class (with one limitation described further below).  You can add mutable get-set properties, or properties that have a different access level to the type itself.  If you want to use positional parameters to initialise a mutable property, you should declare both a positional parameter and a mutable property with the same name, and provide an initialiser for the property which uses the positional parameter, like this:

```
public record Vehicle(string Registration, string Colour)
{
    public string Registration { get; set; } = Registration;
}
```

Record classes can inherit another record class as their base type, and if no inheritance is declared they inherit directly from `object`.  They can't inherit from a class.  Derived record classes that use positional parameters should redeclare all positional parameters that they want to expose in their constructor, and pass them to the constructor of the base type using a syntax similar to calling a base constructor:

```
public record Vehicle(string Registration, string Colour);
public record Car(string Registration, string Colour) : Vehicle(Registration, Colour);
```

If you don't pass values for the base type's positional parameters like this, the properties of the base type will still be visible in the derived type, but will not be intialised.

The generated equality-checking code in record classes will always determine record class instances of different types to be unequal, even if one is derived from the other.  In derived record class types, the equality-checking code compares all properties, whether declared in the derived type or inherited from the base type(s).

The generated code in a record class includes an anonymous cloning method which can't be called explicitly.  Its purpose is to be called in a "`with` expression", which clones an instance of a record, optionally changing some of the positional properties.  The syntax of a `with` expression is similar to a brace initialiser.

```
// assume public record Vehicle(String Registration, string Colour);

Vehicle myCar = new Vehicle("FAB 1", "Pink");
Vehicle resprayedCar = myCar with { Colour = "Blue" };
```

Note that the type of a `with` expression is the same as the declared type of the left-hand operand, not its specific type.  Similarly, implicit calls to the `Deconstruct()` method of a record class call the method of the declared type, not of the specific type.

If you define a record class that has an explicit implementation of any of the generated members described above, the compiler will use the explicit implementation and not generate its own.  As for classes, if you explicitly implement the equality-checking methods, you should also implement `GetHashCode()` with its normal behaviour (ie, instances that compare equal should return the same hash code).  In order for things to work properly, you must give your explicit implementations exactly the same signatures as the generated code would have.  This is particularly important for `PrintMembers()` as its signature depends on several aspects of the type.  In all cases `PrintMembers()` takes a single `StringBuilder` parameter and returns `bool`, but its modifiers vary:

- If a record doesn't define a base type and isn't declared `sealed`, it must be declared `protected virtual`.
- If a record doesn't define a base type but is `sealed`, it must be declared `private`.
- If a record is derived from another record class, it must be declared `protected override`.

If you do get the signature wrong, you will get a compile-time error.  Note that the compiler-provided implementation of `PrintMembers()` in a derived record class will call the `PrintMembers()` implementation of its base type, so bear this behaviour in mind when defining your own.

Similarly, if you provide your own implementation for the copy constructor, it must be declared `private` for sealed record classes and `protected` for unsealed ones.

You can't replace the generated instance cloning method, because it's anonymous.  However, you also can't declare a member named `Clone` inside a record class.

If you implement your own `ToString()` override in a record class, you can declare it `sealed` to prevent the compiler creating its own implementation in any derived record classes.  You can't do that for your own implementations the other potentially-synthesised members, unless the record class itself is `sealed` (in whch case it's moot).  If you try to define, say, a `PrintMembers()` method as `sealed` in a non-sealed record class, you will get a compile-time error.

#### Record structs

Record structs were introduced in C# 10, and are the value-typed equivalent of a record class.  Aside from being a value type instead of a reference type, there are two key differences between records structs and record classes:

- Like a struct, they can't be derived from a user-specified base type and they are all implicitly sealed.
- Whereas record class positional parameters are immutable, record struct positional parameters are mutable by default.

This second point means that the positional parameters of normal record structs are created as get-set properties, not as get-init properties as in a record class.  However, you can also define a `readonly record struct`, in which the positional parameters are made into immutable get-init properties.

In additional to the positional parameter constructor, record structs (both mutable and `readonly`) have a parameterless constructor added by the compiler.  They *don't* have a copy constructor created, as for record structs the `with` operator uses normal assignment.  You can define a copy constructor yourself, but the `with` operator will not call it implicitly as it does for a record class.  Aside from that, record structs have the same generated members, with the same signatures, as a sealed record class.

## More advanced syntax and features

### Anonymous types

We have seen above that&mdash;barring top-level statements&mdash;all C# code is contained within a type, and that all types are named and contained within a namespace.  This is not, however, the full story: it is also possible to define *anonymous types*.  These are well-defined types, and strongly-typed at compile time, but are not accessible by name in your code and have various other limitations:

- They are always classes.
- The only members they can contain are properties.
- The properties they contain are read-only and public.  Their types must be inferrable at compile-time, and they must not be unsafe types.
- They cannot contain methods, fields, or any other kind of member.
- That includes constructors, therefore their only constructor is the implicit parameterless constructor.
- They cannot define inheritance relationships, so are always implicitly derived from `object`.
- Because they're not accessible by name, you can't derive other classes from them.
- For the same reason, they can't be the delared return type or parameter type of a method.  If a method returns an anonymous type it must declare that it returns `object`. 
- They are only defined at the point(s) in the code where they are instantiated.

The definition and instantiation of an anonymous type is similar to a constructor call using brace initialiser syntax, but without the parentheses or type name.

```
var anon = new { Size = 37, Description = "Stuff" };
```

This example defines an anonymous type; it will be given a name by the compiler, and this name will be visible if you disassemble the compiler's CIL output or call `instance.GetType().Name` at runtime, but it is not accessible in code.  The generated type name will probably not be valid as a type name within C# itself, to make sure you don't accidentally declare something which could cause a naming clash.  Visual Studio will give it an auto-generated name such as `'a`, in tool-tips and other on-screen help, but this will probably not be the name that your compiler gives it, which will be something more complex like `<>f__AnonymousType0```1`.  For one thing, the compiler has to ensure that code containing multiple anonymous types uses different names for each distinct anonymous type, whereas Visual Studio does not.

Note the anonymously-typed object is assigned to a variable declared using the `var` keyword for implictly-typed declarations: `var` must be used, as there is no way to declare the variable's type explicitly.

If, elsewhere in the same assembly, another anonymous type is instantiated with the same properties, by both name and type and in the same order, the compiler will ensure that only one anonymous type is defined.  This gives us a form of duck typing: if two anonymous types appear to have the same signature, they will be compatible for assignment.  However, this is a very limited form of duck typing, in that the two types must be completely identical in their property signatures.  Consider the code:

```
var a = new { Size = 37, Description = "Things" };
var b = new { Description = "Stuff", Size = 12 };
var c = new { Size = 8 };
```

In a language using looser duck typing, you might expect to be able to assign between these variables. In C#, no assignments between the three variables are allowed, all six possible permutations of assignment causing compile-time errors.  In particular, even though code that uses variables `a` and `b` could almost certainly be identical, `a` cannot be assigned to `b` or vice-versa, because two unrelated types will be created.

If a property in an anonymous type has the same name as the member being used to initialise it, a slight shorthand can be used in the definition, omitting the `=` sign and everything to the left of it.  This is most useful when wanting to define an anonymous type whose properties are a subset of those of another object.  The following code:

```
var subsetOfX = new { Top = x.Top, Bottom = x.Bottom };
```

can be written more succinctly as:

```
var subsetOfX = new { x.Top, x.Bottom };
```

Anonymous types are commonly-used in LINQ sequences, which will be explained further below.  They are also used by some frameworks such as ASP.NET MVC; for example, in a web API where an API call returns a JSON object which does not otherwise need to have a C# class defined specifically for it, the return data can be generated by serialising an anonymously-typed object.

### Disposables

Although the VES must include a garbage collector responsible for handling memory cleanup of "managed objects" when they are no longer accessible from within running code, there may well be other memory cleanup requirements which it cannot handle by itself.  Code that interfaces with native or unmanaged libraries, or requires external or operating system resources may also have other cleanup requirements.  Code that accesses the filesystem, for example, should try to close open files cleanly; database drivers may require result objects to be closed in order to release server-side resources (this is particularly an issue with connections to Microsoft SQL Server or Azure SQL).  In most cases, this is accomplished most cleanly in C# by utilising the disposable pattern.  This is a standardised pattern which provides an interface that the VES garbage collector can hook into and use to ensure that external resources are always cleaned up before the garbage collector destroys an object, but also that the code can clean those resources up earlier if suitable, without confusing the garbage collector later.

It might be helpful at this point to run through a very brief and high-level description of how the VES garbage collector works, conceptually.  It classifies all objects as belonging to one of three *generations*, numbered 0, 1 or 2.  As we've said the only way to allocate memory to create a reference object in C# is ultimately via the `new` operator, and in the vast majority of cases when this happens, the object is marked as belonging to Generation 0.  The exceptions are "large objects", bigger than about 84kB in size, which are allocated from a separate "large object heap" and marked as Generation 2.

The details around exactly when the garbage collector runs, exactly how it behaves, and whether other execution is suspended when it does run, vary from implementation to implementation and version to version, but at the highest level some things are common across all implementations of the VES and all configurations.  A garbage collection run also has a generation number; a GC run of generation X will collect garbage from generation X and all lower generations, so because of this Generation 2 GC runs are also called "full garbage collection runs".  During the run, the garbage collector will destroy objects that are no longer accessible to running code.  Surviving objects in generations 0 and 1 will each then be promoted into the next generation up.  In other words, after a garbage collection run, Generation 0 will be empty; all Generation 0 objects will either have been destroyed or promoted; and after a Generation 1 or 2 GC run, none of the objects in Generation 1 beforehand will be in Generation 1 afterwards for the same reason.  Long-lived objects quickly rise to Generation 2, where they will stay until destroyed; but in a typical program most objects will be destroyed within the first few GC runs to encounter them.  Typically, the garbage collector will carry out frequent generation 0 runs, less frequent generation 1 runs, and generation 2 runs will be the least frequent of all.

We have already mentioned defining a finalizer method on a class.  When the garbage collector destroys an object, it checks to see if such a method exists: if it does, the garbage collector may need to ensure that the finalizer runs.  However, to avoid making garbage collections slower and to ensure that the garbase collector does not need to run user-defined (and hence relatively unpredictable) code during a GC run, this is done by posting a reference to the object on the garbage collector's "finaliser queue".  The garbage collector generally runs through the finaliser queue in a separate thread, at a later time.  So that the object stays available until its finaliser is run, anything posted to the finaliser queue will be kept in memory until its finaliser has been run, and will be promoted up to Generation 2 if it wasn't there already.  Its memory will finally be reclaimed on the first full garbage collection run after the finalizer method has completed.

You might think, from the above, that this makes everything straightforward.  If your code opens a database connection, or opens a file, just define a finalizer in the relevant class and make sure that the finalizer cleanly closes everything left open.  Indeed, this would work to some extent.  However, it's not the recommended pattern, in part because there is no guarantee how long it might take between the garbage collector determining that an object is no longer reachable, and the object's finalizer being run.  If the object is holding a limited resource such as, say, an open database connection or an open file handle, this could be problematic: it could lock up resources that don't really need to be reserved.

The solution to this is to use the dispose pattern.  At the cost of adding some boilerplate code to classes which need to implement it, it provides a pattern which makes it straightforward to clean up resources before the garbage collector identifies an object as unreachable, and signals to the garbage collector that these objects do not need to be held around to be finalized, whilst also functioning properly when called from code that does not follow the pattern.  Moreover, the language itself provides syntactical support to assist with using the pattern.

The disposable pattern essentially consists of three things:

- The `System.IDisposable` interface.
- The boilerplate which should be used to implement it.
- The `using` keyword, when used as a statement or to prefix a local variable declaration.  This isn't related to `using` directives, discussed earlier.

The `System.IDisposable` interface itself is relatively straightforward: it contains a single method, which returns `void` and takes no parameters, named `Dispose()`.  However, choosing to implement this interface should be taken as a convention that has certain implications: that the class has resources which need to be cleaned up, that the `Dispose()` method will do this, and that the method will behave in certain conventional ways.  In particular, the `Dispose()` method must be written so that it can be called more than once, although the second and subsequent invocations should be no-ops.  In return, it is perfectly legitimate for other methods of the object to stop functioning once `Dispose()` has been called, and there is a standard type, `System.ObjectDisposedException`, which methods can throw if they are called after `Dispose()` has been called on their instance.  The standard boilerplate for implementing `Dispose()` does properly handle multiple invocations, adds a member field to indicate that an instance has been disposed, and also, once resources have been cleared up, registers the object with the garbage collector as one that does not need finalization even though it has a finalizer method.  

In general, all classes which are responsible for managing fields or properties which are of `IDisposable` types should themselves implement `IDisposable`, so that they can dispose of the fields in question.  You should also implement `IDisposable` if your class holds references to any sort of external or operating system resource.  In general, you should find that the .NET API for accessing such resources will do so through `IDisposable` objects, so the two requirements dovetail quite naturally.  However, there might be cases in which your class holds a reference to a disposable type but doesn't want to dispose it: for example, an open file which might still need to be used.  Potentially, you might need to have some means to indicate whether the class should dispose of members or not when it is itself disposed.  For example, the `System.IO.Compression.ZipArchive` class (used to access the contents of ZIP files) always takes a `System.IO.Stream` constructor parameter which contains (or will contain) the underlying raw data of a ZIP archive, and has some constructors with a `bool` parameter to indicate if the underlying stream should be held open when the archive itself is disposed.  If, say, you are constructing a ZIP archive in memory you can pass a `MemoryStream` to a `ZipArchive` constructor, but must instruct it not to dispose the `MemoryStream` when disposing the `ZipArchive`&mdash;otherwise your in-memory ZIP file will be thrown away before you can do anything with it. 

There are a few slightly different forms of the dispose pattern, depending on whether or not your class accesses unmanaged resources (or external code) directly or not, and whether or not your class is derived from another class which also follows the dispose pattern.  If your class is not derived from a class that implements the pattern, and it is `IDisposable` because it manages resources which are themselves managed `IDisposable` resources, then the outline of the pattern is as follows:

- Create a `bool` field to track whether or not the object has been disposed.
- Create a `protected virtual` method with the signature `void Dispose(bool disposing)` which carries out the disposal work.
- Implement the `IDisposable` interface by creating a `public void Dispose()` method which calls the protected override that actually does the work, and then registers the object with the garbage collector as one which does not require finalization.

The boilerplate code for the above looks something like this:

```
public class DisposableClass : IDisposable
{
    private bool _disposed = false; // Has this instance been disposed?

    private IDbConnection _dbConnection; // An example of an IDisposable field.

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this); // Notify the garbage collector that finalization is not required.
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed)
        {
            return;
        }

        if (disposing)
        {
            // This is where our cleanup work goes.
            _dbConnection?.Dispose();
        }

        _disposed = true;
    }
}
```

Your `Dispose(bool disposing)` method should also, to be polite, set any large fields to `null` to give the garbage collector a chance to destroy them&mdash;although this won't be possible if they've been declared `readonly`.

If your class is `sealed` then the `Dispose(bool d)` method has to be `private` rather that `protected virtual`, but it still helps to keep the two methods separate for clarity.

If you have unmanaged resources that need to be cleaned up, the recommended pattern is to use one of the classes derived from the abstract `System.Runtime.InteropServices.SafeHandle` class to wrap the unmanaged resource.  On the Windows platform there are a number of classes in the `Microsoft.Win32.SafeHandles` namespace to wrap things like OS-level filehandles, registry handles, and suchlike; these provide a lightweight way to wrap the external resources in managed `IDisposable` objects that will take care of finalization for you.  However, if you don't want to do this, you can also do the following:

- Add the code to clean up your unmanaged resources to the `void Dispose(bool d)` method immediately before `_disposed = true;`, so that it is called regardless of the value of the parameter if the method has never been called previously.
- Create a one-line finalizer method that calls `Dispose(false)`.

You should still call `GC.SuppressFinalize(this)` as before; the finalizer will only be called if your class is used from code which does not itself use the dispose pattern correctly and therefore never calls `Dispose()`.  We'll discuss how you can ensure that you do call `Dispose()` correctly below.

If you are writing a class which is derived from another disposable class, and which follows the pattern correctly, you do not need all of the boilerplate.  Indeed, if the derived class does not have any additional resources for clean-up, you don't need to do anything at all.  If it does, all you need to do is create another field to track disposal, and override the protected `Dispose(bool d)` method, as shown here:

```
public class DerivedDisposableClass : DisposableClass
{
    private bool _disposed = false; // Note: separate to the _disposed field of the base class.

    private StreamReader _reader; // Another arbitrary example IDisposable resource.

    protected override void Dispose(bool disposing)
    {
        if (_disposed)
        {
            return;
        }

        if (disposing)
        {
            _reader.Dispose();
        }

        _disposed = true;
        base.Dispose(disposing);
    }
}
```

Note the tail call to the base method: this is vital to ensure that private members of the base class are disposed properly, which is also why the base and derived classes have separate `_disposed` fields, and why the `Dispose(bool disposing)` method must override the base implementation instead of hiding it, so that it is called first even if it's been accessed through a reference to the base type.  If your derived class also has unmanaged resources which need cleaning up (and cannot be wrapped in a `SafeHandle` instance) then add the necessary code in the same way as for the base class: put the cleanup code before `_disposed = true;` and add a finalizer that calls `Dispose(false)`.

How do you know when you need to implement this type of pattern?  In general, if the class you are inheriting from implements `IDisposable` and defines a `protected virtual void Dispose(bool d)` method, you should assume that it is using the dispose pattern as described here, and therefore you should override the `Dispose(bool d)` method as per this example.

The above pattern is straightforward enough&mdash;and there is more information and examples of more complex cases on the .NET documentation website&mdash;but what is the best way to ensure that you do always call `Dispose()` on each `IDisposable` object before it falls out of scope?  The answer to this is: with sensible use of the `using` keyword.  The `using` statement declares and initialises a variable; when the variable falls out of scope at the end of the block, its `Dispose()` method will automatically be called.  Traditionally, the `using` statement required its own block of code; since C# 8.0, the "simple `using`" statement can be used to declare a variable which will be disposed at the end of the enclosing block.

In use, `using` looks line this:

```
// Traditional style
using (IDbConnection conn = GetOpenDatabaseConnection())
{
    // code using the "conn" variable goes here.
}
// "conn" is out of scope and has been disposed

// "Simple" style - C# 8 and newer.  This variable's scope will be the same as a normal declaration.  conn8 will be disposed when it goes out of scope.
using IDbConnection conn8 = GetOpenDatabaseConnection();
```

The "simple style" is equivalent to a block using statement in a method with no further statements after the end of the block, which is a very common use case if your code is divided into suitably-sized methods.

The important thing here is that the `conn` variable is declared and instantiated at the start of the block&mdash;we have hidden the details of how to open our connection behind a method, for the purposes of this example.  The connection can then be used as long as the variable remains in scope.  When the declared variable falls out of scope, `conn.Dispose()` is *automatically* called for us, through boilerplate code inserted into our CIL by the compiler.  Essentially, the above code is functionally equivalent to:

```
{
    IDbConnection conn = GetOpenDatabaseConnection();
    try
    {
        // The contents of the using block would be here.
    }
    finally
    {
        conn.Dispose();
    }
}
```

Bear in mind that with a `using` block, `Dispose()` is called when the *declared variable* goes out of scope, which may well be before the object it refers to becomes unreferenced.  You must always be careful about leaking references which will be automatically disposed to the outside of their `using` block.  An obvious example is this slightly contrived case:

```
IDbConnection savedConnection;
using (IDbConnected conn = GetOpenDatabaseConnection())
{
    savedConnection = conn;
    // do some database work.
}

// savedConnection refers to the object which was already Dispose()d at the end of the previous using block.
// Therefore, this will probably not behave very well.
savedConnection.BeginTransaction();
```

However, there are more subtle cases, particularly when an `IDisposable` declared in one `using` block is passed by injection into another `IDisposable` which takes responsibility for it.  Consider the following skeleton code for reading from a file:

```
using (FileStream stream = new FileStream(fileName, FileMode.Open))
using (StreamReader reader = new StreamReader(stream))
{
    // Process file.
    ReadFileIntoMemory(reader);
}
```

This is idiomatic .NET file-reading code, and incidentally shows what was a fairly common code formatting style when using nested `using` blocks, before the "simple using statement" was introduced: if the only code contained within one `using` statement is a nested `using` statement, some coding styles will use neither indentation nor braces for the contents of everything other than the innermost `using`, to avoid excessive indentation.  This formatting style is very easily confused with a simple using statement: the only differences are the parentheses and the lack of a `;` at the end of the first using statement, which means that this is a block using statement whose block consists solely of the following using statement.

More importantly, note that according to the documentation for the `StreamReader` class, it releases all of its resources when `Dispose()` is called.  This will include the `FileStream` from the outer block, because this was injected into the `StreamReader` as a resource.  This means firstly that `stream.Dispose()` will be called twice, once from `reader.Dispose()` at the end of the inner block; then again at the end of the outer block.  This is fine; because as we said earlier a `Dispose()` method must always be coded to be able to handle multiple calls.  However, it does mean that we have to be careful not to accidentally write code like this:

```
using (FileStream stream = new FileStream(fileName, FileMode.Open))
{
    using (StreamReader reader = new StreamReader(stream))
    {
        // Process file.
        ReadFileIntoMemory(reader);
    }
    // Attempt to do something else with the file.
    stream.Seek(0, SeekOrigin.Begin);
    ReReadStream(stream);
}
```

Here, we are trying to call `stream.Seek()` after `stream.Dispose()` has already been called.  The result will be that the `stream.Seek()` call throws `ObjectDisposedException`, even though we are still within the `using` block for `stream` itself.

Cases like this can lead you to think that `using` can be too brittle and restrictive for a lot of cases.  It's true that in situations where you need to keep a resource on hand for a long period of time then `using` is not always the most suitable way to go, although occasionally libraries will do what they can help you.  For example, the ADO.NET database library for SQL Server by default maintains a database connection pool.  Calling `SqlConnection.Dispose()` returns the connection to the in-memory connections pool and does not close it immediately, so a call to open another identical connection a few milliseconds later will reuse the extant connection rather than imposing the overhead of opening a new one.  Dependency injection frameworks, such as the one provided with .NET Core, should automatically recognise when `IDisposable` instances fall out of scope and call `Dispose()` on them at the appropriate time in the DI lifecycle&mdash;the .NET Core DI framework certainly does, but if you are using .NET Framework with a third-party DI library, check your documentation.  Other frameworks you may be using may also provide hooks to call `Dispose()` at appropriate times; for example, ASP.NET Core allows you to associate `IDisposable` objects with an HTTP request so that `Dispose()` will be called when that request has completed (the method you need here is `HttpContext.Response.RegisterForDispose(IDisposable disposable)`).  In general, though, try to think about how you can utilise `using`, because it is the most straightforward and reliable way of making sure that every `IDisposable` object gets its `Dispose()` method called.

### Lambdas and delegates

In other languages, you may have come across concepts such as function pointers, function references, anonymous functions or method references.  The equivalents in .NET are delegate types and lambda expressions.  The functionality of this area of the language is one which has changed significantly since C# 1.0, so in this area more than most there are a number of ways to do the same task; however, in general, the language has moved over time from relatively static and inflexible constructs to more flexible ones.

Since its introduction, the language has included the concept of *delegate types*.  A delegate defines a type which represents a reference to a method with a particular signature.  For instance, a delegate type to represent methods which carry out integer arithmetic operations might be declared as follows:

```
public delegate int ArithmeticOperation(int a, int b);
```

Note that this looks rather like an abstract method declaration; but the `delegate` keyword indicates that it is actually declaring a type, whose name is `ArithmeticOperation`, and which is a delegate type.  A variable can be declared to be of this type, and any static or instance method, on any class or of any name and of any access level, whose type signature matches that of the delegate, can be assigned to such a variable.  They can also be passed as parameters, and then can be called like a method.

```
public class Example
{
    public int Add(int a, int b) { return a + b; }

    public int Multiply(int a, int b) { return a * b; }

    public int Apply(ArithmeticOperation op, int a, int b)
    {
        if (op != null)
        {
            return op(a, b);
        }
        return 0;
    }

    public int DoSomeThings()
    {
        ArithmeticOperation operation = this.Add; // Note, no parentheses.
        Console.WriteLine(Apply(operation, 5, 6));
    }
}
```

The examples above are a bit contrived, but hopefully they demonstrate the principles of the syntax.  Note that a delegate representing an instance method is a reference to the method on a *specific* instance, and calling it will call that instance's method; you do not need to bind the reference to an instance separately.

Delegate type references themselves also have an `Invoke()` method.  From C# 6 onwards, using `Invoke()` in combination with the null-conditional member access operator `?.` can make safe calls to delegates more succinct and thread-safe.  The `Apply()` method in the previous example could be rewritten in one line as:

```
public int Apply(ArithmeticOperation op, int a, int b)
{
    return op?.Invoke(a, b) ?? 0;
}
```

Delegates can be composed into *invocation lists* using the addition operators.  If, for example, you have two delegate variables of the same type named `a` and `b`, the expression `a + b` returns a delegate which is of the same type, but which is a *multicast delegate*; when invoked, `a` and `b` will each be called, in that order.  If they have a return type, the return value of the multicase delegate will be the return value of the final delegate in the invocation list&mdash;`b` in our example.  This can be very useful in situations where a method takes a delegate parameter as a callback, because it enables the calling code to transparently specify multiple callbacks, without the called code needing to know or care how many callbacks have been passed (as long as the parameter is not `null`).  The `+=` operator can also be used to append a delegate to another, as you might expect: `a += b` will append `b` to `a`'s invocation list.

You can also, incidentally, remove a delegate from a multicast delegate invocation list.  Given that delegates are added to invocation lists using the addition operators, you will hopefully not be too surprised to find that they are removed using the subtraction operators.  If a delegate named `multicast` is composed of delegates `a` and `b`, then `multicast -= a` will remove `a` from the `multicast` invocation list, making it equivalent to calling `b` alone.

In C# 1.0 it was often necessary for developers to define their own delegate types to match whatever particular type signatures they needed.  With the introduction of generic types in C# 2.0, the framework also supplied generic delegate types which can avoid the need to define your own.  The number of these types has increased over time; but from .NET Framework 4.0 and .NET Standard 1.0 onwards they have supported methods with up to sixteen parameters, which should cover the vast majority of cases.

These types are `Action<...>` for void methods and `Func<...>` for methods with non-void return types.  The framework defines `Action<T>`, `Action<T1, T2>`, and `Func<TResult>`, `Func<T1, TResult>`, `Func<T1, T2, TResult>` and so on up to sixteen different type parameters (seventeen including the return type of `Func<T1, ..., T16, TResult>`).  Our example `ArithmeticOperation` delegate type is not needed: it would be covered by `Func<int, int, int>`.  The only benefit to still defining your own delegate type is clarity when reading the code, as your delegate type name can encapsulate the purpose of the type much better than something arbitrary like `Action<string, string>` or similar.

C# 2.0 also introduced anonymous methods with the use of the `delegate` operator, but this is now little-used and is not recommeded in new code.  The example here is provided so that you know what one looks like, should you encounter one.

```
Func<int, int, int> multiply = delegate (int a, int b) { return a * b; };
```

C# 3.0 introduced *lambda expressions*, a more concise way to define anonymous methods.  As a lambda expression, the previous example looks like this:

```
Func<int, int, int> multiply = (a, b) => a * b;
```

Lambda expressions are defined using the `=>` (or "fat arrow") operator; the left-hand side is the parameter list and the right-hand side is the body of the method.  This example is an *expression lambda*, where the body consists of a single expression.  Lambda method bodies can also consist of a block of code, like an anonymous delegate method; these are called *statement lambdas*.  Note that the parameters are contained in parentheses: if the lambda has exactly one parameter the parentheses are sometimes not required (see below for situations where they are).  A lambda with no parameters is defined using an empty pair of parentheses: `() => ...`.

```
Func<int, int> square = a => a * a;
Func<int> random = () => (new Random()).Next(); // probably a bad example
```

In the above examples, note that the lambda parameters do not appear to be typed.  Lambdas are&mdash;like most of C#&mdash;strongly typed, but in all of the cases above the compiler can infer the parameter types from the type of the variable that the lambda is being assigned to.  If this is not the case, the types can be specified explicitly.

```
// Because we are now telling the compiler to infer the type of the multiply variable, we have to specify
// the types of the lambda parameters.
var multiply = (int a, int b) => a * b;
```

Lambdas can either have all parameters explicitly typed or all parameter types inferred; but a given lambda can't mix and match.

From C# 10 onwards, a lambda can have its return type explicitly specified by putting the type before the parameter list.

```
var square = int (int x) => x * x;
```

If you specify either the return type or parameter type, a single-parameter lambda must put parentheses around the parameter.

In general, lambdas can access most variables in the scope in which they are defined, referred to as *outer variables*, and the access of an outer variable by a lambda is called *capturing*.  A declared-but-undefined variable, or an `in`, `ref` or `out` parameter variable, cannot be captured by a lambda.  When a lambda captures a variable, its value is stored so that it can still be used by the lambda after it falls out of scope; however, it does not prevent normal operation of `using` blocks.  Consider the following example, adapted from the examples above on the `using` statement:

```
Action dumpfile, printLength;
string fileName = "file.txt"; // replace this with the path to a text file if you want to try this out.
using (FileStream stream = new FileStream(fileName, FileMode.Open))
using (StreamReader reader = new StreamReader(stream))
{
    long streamLength = stream.Length;
    printLength = () => Console.WriteLine(streamLength);
    dumpFile = () => Console.Write(reader.ReadToEnd());
}

Console.WriteLine("Files are now closed.");
printLength();
dumpFile();
```

When this is run, the `printLength()` call will succeed and print the value of the captured `streamLength` variable, even though that variable has gone out of scope.  However, the `dumpFile()` call will throw an exception, because the captured `reader` variable was disposed when it went out of scope.

Lambda outer variable capture captures the variable itself, not its value.  This can be seen by a small modification to the above example:

```
Action printLength;
string fileName = "file.txt"; // replace this with the path to a text file if you want to try this out.
using (FileStream stream = new FileStream(fileName, FileMode.Open))
using (StreamReader reader = new StreamReader(stream))
{
    long streamLength = stream.Length;
    printLength = () => Console.WriteLine(streamLength);
    streamLength = -1;
}

Console.WriteLine("Files are now closed.");
printLength();
```

The output from this example will be:

```
Files are now closed.
-1
```

The lambda is printing the value of `streamLength` at the time of execution, not at the time of capture.

From C# 9 onwards a lambda can be declared as `static`.  This prevents it from capturing any instance members as outer variables, and gives the compiler the option&mdash;which it doesn't have to take&mdash;of compiling it as a static method.  A static lambda can capture static members (or `const` members, as they are implied static) of its containing type, just as a static method can use static members.

```
Func<int, int> square = static (x) => x * x;
```

Lambda expressions, at their heart, often compile down to the same CIL as delegates.  However they are rather more concise and flexible in their application, and as a result are seen in modern C# code much more often than the older syntax.  In particular they are often seen in LINQ code; probably the majority of LINQ methods have at least one overload which takes a `Func<T1, TResult>` parameter.  We will discuss LINQ in more depth below, once we have discussed its other key building block, the iterator.

Expression lambdas can also compile to another kind of object, the *expression tree*.  Expression trees can also be generated programmatically to create any kind of expression that C# can represent, and this can then be executed.  However, statement lambdas cannot be converted into expression trees.  When compiling, the compiler determines from context whether to compile an expression lambda into an anonymous method, or into code that generates an expression tree.  Expression trees can be examined at runtime: for example the Entity Framework library takes expression trees as parameters to many of its methods, and converts the .NET expression trees into SQL queries. 

### Iteration and iterators

The concept of iteration has been a key feature of C# since its creation, due to the `foreach` statement described above.  It essentially revolves around two core interfaces:

- `System.Collections.Generic.IEnumerable<T>`, which defines objects whose members can be enumerated.
- `System.Collections.Generic.IEnumerator<T>`, which defines objects that carry out enumerations.

The interfaces listed above are the generic forms introduced with generic types in C# 2.0; they both inherit from older non-generic versions, `System.Collections.IEnumerable` and `System.Collections.IEnumerator`.  However, the basic principles are the same; the primary difference is that the generic form enables iterator use to be type-safe.

The `IEnumerable<T>` interface defines one key method: `IEnumerator<T> GetEnumerator()` (and also inherits the non-generic form from `IEnumerable`).  The `IEnumerator<T>` interface is disposable, and defines two methods and one property: `bool MoveNext()`, `void Reset()` and `T Current`; the `Current` property also has a non-generic form declared as `object`.

`IEnumerable<T>` implementations can be very simple indeed: the current .NET Core implementation of `List<T>.GetEnumerator()` uses method expression syntax to reduce its implementation to two lines of code:

```
IEnumerator<T> IEnumerable<T>.GetEnumerator()
    => new Enumerator(this);
```

The magic is clearly all in the enumerator itself.  In the case of `List<T>` this is defined as a nested `struct`, enabling it to have access to the `List<T>`'s internals.

The basic principles of iteration are that the enumerator object has some concept of "the current item", and this is the value returned by the `Current` property.  `MoveNext()` changes the current item to be the "next" in the enumeration, whatever "next" might mean in this context, and `Reset()` resets the enumerator back to its default state, if this is possible.  It might not be.

Note that these operations are very simple, and not guaranteed to be reversible.  As per the previous paragraph, `Reset()` might not work.  The enumerable's members do not necessarily have to have a fixed index, and the enumeration might not necessarily be a reproducible operation; if it is, it might not return items in the same order.  Another basic limitation on the process is that enumeration is allowed to assume that the underlying members are fixed, at least for the duration of the enumeration.  Consider the following code:

```
void BreedSomeBacteria(List<Bacteria> colony)
{
    foreach (Bacteria cell in colony)
    {
        colony.Add(cell.Divide());
    }
}
```

This will&mdash;other than for the trivial case of an empty list&mdash;throw an exception.  The call to `List<T>.Add()` will modify the list.  The foreach statement hides a call to `List<T>.Enumerator.MoveNext()` which will detect that the `List<T>` has been modified since the `List<T>.Enumerator` was created, and throw an `InvalidOperationException` as a result.

When an enumerator is first created, its `Current` property should be in an invalid state, and accesses of the property should throw `InvalidOperationException`.  The first invocation of `MoveNext()` should&mdash;if the enumeration contains any items&mdash;move it on to the first valid item.  `MoveNext()` returns `true` if the call was successful&mdash;in that an item was available to move to&mdash;and `Current` is now that item; if `MoveNext()` returns `false` it means there are no more items, and the enumerator's `Current` property is once more in an invalid state.  In general, if you want to move through the whole enumeration, keep calling `MoveNext()` until it returns `false`, which means you've gone off the end of all of the items.  In keeping with the simplicity of the interface, there is no way to tell whether or not `MoveNext()` will succeed without calling it.

Incidentally, if you're thinking "but hang on a minute there, we *do* know how long an array is, or how long a `List<T>` is"&mdash;yes, we do.  However they also implement more advanced interfaces such as `ICollection<T>` and `IList<T>` which give you a few more guarantees than a plain `IEnumerable<T>`, such as a known length and stable item position (items always having the same position in the data, in other words).  Here though we're working at the lowest level, and we don't have those guarantees.  That does mean we can, say, start iterating over a large data set without having loaded it all, or without having received it over the network.

You can see from the above, now, how we could implement the `foreach` keyword if it did not exist.  The loop:

```
foreach (var x in items)
{
    // ...
}
```

could be implemented as:

```
{
    var enumerator = items.GetEnumerator();
    while (enumerator.MoveNext())
    {
        var x = enumerator.Current;
        // ...
    }
}
```

This is equivalent to a `foreach` loop in C# 5 or later.  We discussed earlier that from C# 5 onwards, a breaking change was made, which affected the behaviour of `foreach` loops with respect to lambda variable capture.  Now that we have discussed that topic, we can see what the exact difference in the two behavioural patterns is.  Prior to version 5, the equivalent `while` implementation of a `foreach` looked like this:

```
{
    // assume items is of type IEnumerable<T>
    var enumerator = items.GetEnumerator();
    T x;
    while (enumerator.MoveNext())
    {
        x = enumerator.Current;
        // ...
    }
}
```

Note that in this case the loop variable `x` is declared once, outside the loop, and is reassigned on each loop iteration.  Consider what happens if `x` is then captured by a lambda expression, and recall we said earlier that a lambda expression captures the variable, not its value.  With the "original" version of a `foreach`, if the lambda is executed after the loop has completed the value of a captured loop value will always be its value on the final iteration of the loop; with the "modern" version, the lambda expression created on each iteration of the loop will capture a different variable.

As we said above, enumeration is not guaranteed to be reversible, resettable or repeatable: running the enumeration again is not guaranteed to reproduce the same results in the same order, although in many cases it will do.  In other words, once `IEnumerable<T>.GetEnumerator()` has been called, there is no guarantee that you will be able to call it a second time.  In many cases&mdash;for example, if the enumerable is actually a `List<T>` or an array&mdash;there is no problem at all, and enumerating a second time will return the same results (barring changes to the underlying data of course).  However, good development tools should give you a warning if you do enumerate over the same enumerable twice, in case this may not be possible.  If you do need to be able to enumerate over the same data twice, you can convert it to one of the more fully-featured types that allows multiple enumeration by calling the `ToArray()` or `ToList()` method.  Those methods are part of the LINQ API, which we'll discuss next.

### LINQ

LINQ, or Language-Integrated Query (nobody actually uses the full name) was introduced as a key component of .NET Framework 3.5 and C# version 3.0.  The original concept behind it was to provide a mechanism to embed set-oriented data-querying statements (such as those provided in SQL) within .NET languages.  With this in mind, C# 3.0 contained a whole new SQLalike syntax for defining queries as expressions.  However, this was syntactic sugar for a much more "normal" fluent API consisting of extension methods on the `IEnumerable<T>` interface, and as a result few developers really use the SQL-style syntax.  This document will concentrate on the fluent API, but I will describe the SQL-like syntax so that, at least, you can recognise it when you see it.

#### LINQ basic principles

Under the hood LINQ officially consists of a number of different "providers" such as "LINQ to Objects" for in-memory operations, "LINQ to XML" for processing XML document nodes, and other libraries which can act as LINQ providers such as the Entity Framework ORM.  However, all provide almost the same API, the LINQ to Objects library providing extension methods for `IEnumerable<T>` and the other providers providing extension methods for more specific types derived from `IEnumerable<T>`.  In one sense this makes development very easy because the code to carry out conceptually related operations always looks the same regardless of which provider is actually being called.  However it can lead to confusion when methods which have the same name across different providers have slightly different signatures or different restrictions on how they can be called depending on context.  I'll try to call out some of the major restrictions you might come across, but this can in some cases result in frustrating bugs that only occur at runtime, or occur in real-world situations but not when the data source has been mocked out.

The most commonly-used LINQ method is probably `Select()`, which represents a projection or map operation.  Its full signature, for the LINQ to Objects version which operates on `IEnumerable<T>` is basically:

```
public static IEnumerable<R> Select<S,R>(this IEnumerable<S> source, Func<S,R> selector)
```

Here I've expressed the type parameters as `S` for source and `R` for result, for brevity; in code, it's very rare for you to have to specify the type parameters manually, because in almost all cases they can be inferred by the compiler.  The purpose of the method is to take a source sequence and transform each element by applying a projection function called `selector` to each one.  Its name is taken from the SQL `SELECT` statement, but it is not meant to imply that `selector` selects which elements appear in the output and which do not; the number of elements in the result will be the same as in the output, with a one-to-one mapping between them.

As this is an extension method, the normal way to call it is as if it were an instance method of an `IEnumerable<T>` type, with a lambda as its parameter.  Here's an example of a `Select()` call that doubles a sequence of numbers:

```
double[] values = new[] { 0.6, 7.2, 138.94, -8642.8 };
var output = values.Select(x => x * 2);
```

As I said, we don't need to specify that we are calling `Select<double, double>()` and equally we don't need to specify the type of the lambda parameter.  One thing to note, though, is that although our input type is `double[]`, the output type is not&mdash;it's `IEnumerable<double>`, so you are a little more limited in the API available to you.  LINQ comes with helper methods `ToList<T>()` and `ToArray<T>()` which will construct sequences of either of those types from an `IEnumerable<T>` to make life easier&mdash;although on the other hand, developers often fall into the trap of wastefully calling these when they don't need to.  The specific type returned by `Select()` will be an internal type of the LINQ provider, and it will likely be a type that encapsulates the arguments passed to it but defers actual execution of the lambda until it is enumerated over, to reduce the need to allocate intermediate storage for large sequences&mdash;we'll discuss the consequences of this later.

```
double[] values = new[] { 0.6, 7.2, 138.94, -8642.8 };
double[] arrayOutput = values.Select(x => x * 2).ToArray();
List<double> listOutput = values.Select(x => x * 2).ToList();
```

Obviously there are many other things you could do here.  You could change the type of the members of a sequence, or format them as strings.

```
IEnumerable<string> output = values.Select(static (x) => x.ToString());
```

The `static` modifier on the lambda is only allowed in C# 9 and later.

Because the second parameter to the method is a `Func<S, R>` you could have an arbitrarily complex anonymous method in there, but be careful that this can result in somewhat difficult-to-read code:

```
IEnumerable<double> output = values.Select(x => {
    double y = GetSomeValueFromX(x);
    if (y > 0)
    {
        double sum;
        for (int i = 0; i < (int)y; ++i)
        {
            sum += GeneratedValue(x);
        }
        return sum;
    }
    return -1 * x;
});
```

The code in the above example is just some arbitrary nonsense, but hopefully it shows how complex, potentially, the sort of thing that can grow up if too much logic is put inside an anonymous method.  If you find yourself coding something like this, you might want to try to think of a better way to do it.

One common use of `Select()` is to only return the specific properties of an object you are interested in.  This can be particularly useful when writing a Web API: your repository layer might return a sequence of objects, but your API code needs to ensure that the serialised response only returns specific properties of those objects.  This is also one of the most common uses of anonymous types:

```
IEnumerable<Egg> rawEggs = LoadEggs(queryParams);
var eggData = rawEggs.Select(static (e) => new { e.Colour, e.Diameter });
return SerialisationHelper(eggData);
```

In this case, we have to use `var` to declare the output data type, as it will be a partially anonymous type; as we discussed above, Visual Studio will describe it as something like `IEnumerable<'a>` but the actual name of the anonymous type will be something compiler-defined and not usable in code.

There's a second `Select()` overload which has the following signature:

```
public static IEnumerable<R> Select<S,R>(this IEnumerable<S> source, Func<S,int,R> selector)
```

The difference is that the lambda method has a second parameter.  When the lambda of this variant is called, its position in the sequence will be passed as the second parameter.  If the specific type of the sequence is something like `List<T>` then this will be the same as the item's index; however, it is also passed for types that do not have a numerical index defined, in which case it will just be the position of the item in this iteration.  You can use it for any situation where it is useful information to have to hand:

```
var formattedOutput = data.Select(static (x, i) => $"{i}: {x}");
```

I said above that the object returned by `Select()` will almost certainly not be an array or a `List<T>`, but will actually be some sort of class that stores the data needed to execute the operation when requested&mdash;when the object is enumerated, in other words.  This can be demonstrated with code like the following:

```
class Program
{
    private static void OutputDataSlowly(IEnumerable<string> data)
    {
        foreach (string str in data)
        {
            Console.WriteLine(str);
            Task.Delay(1000).Wait();
        }
    }

    static void Main(string[] args)
    {
        string[] data = new[] { "One", "Two", "Three", "Four" };
        IEnumerable<string> timestampedData = data.Select(x => $"{DateTime.Now.ToString("HH:mm:ss.fff")} {x}");
        Console.WriteLine("First block");
        OutputDataSlowly(timestampedData);

        timestampedData = data.Select(x => $"{DateTime.Now.ToString("HH:mm:ss.fff")} {x}").ToArray();
        Console.WriteLine("Second block");
        OutputDataSlowly(timestampedData);
    }
}
```

This program will produce rows of output slowly, about one line a second.  The output will look something like the following:

```
First block
09:54:24.915 One
09:54:25.939 Two
09:54:26.943 Three
09:54:27.943 Four
Second block
09:54:28.945 One
09:54:28.945 Two
09:54:28.945 Three
09:54:28.945 Four
```

Note how in the first block the timestamp increments in line with the delay between printing each row, as the lambda expression is not called until the code implementing the `foreach` loop calls the enumerator's `MoveNext()` method.  In the second block, however, every row has the same timestamp (or if they do differ, they will only differ marginally).  The difference is the `ToArray()` call.  This is one of a number of methods referred to as *materialisation methods* because they force the earlier steps in the chain to be evaluated.

The core LINQ library defines an `IQueryable<T>` interface which extends `IEnumerable<T>`.  The purpose of this interface is to be implemented by providers, to represent things such as database tables or structured documents that can be queried.  Each `IQueryable<T>` implementation is responsible for translating LINQ method calls into whatever the underlying data source provider requires: for example, Entity Framework generates `IQueryable<T>` implementations representing data tables, and converts LINQ methods (or chains of LINQ methods) into SQL queries that are sent to the database for execution when the results have to be materialised.

In theory, LINQ should enable you to write almost-identical code regardless of the underlying provider.  However, this is not quite the case in practice.  Firstly, the signatures of the `Select()` calls on `IQueryable<T>` are not exactly the same as those we have seen on `IEnumerable<T>`.  They are:

```
private static IQueryable<R> Select<S, R>(this IQueryable<S> source, Expression<Func<S, R>> selector);
private static IQueryable<R> Select<S, R>(this IQueryable<S> source, Expression<Func<S, int, R>> selector);
```

Note that the second parameter in each case is an expression tree, not a delegate.  This means that code that uses an *expression lambda*, like most of our examples above, will be fine.  However a statement lambda cannot be converted to an expression tree, so the call will instead resolve to `IEnumerable<T>.Select()`.  This could potentially have a behavioural effect on the code: for example, it could mean that a data filtering step you intended to be done by the database server actually gets done within your process, greatly increasing the amount of data transferred over the network.  It will hopefully be clearer how things like that can happen when we have discussed more of the methods available in the LINQ API.

The second difference between `IEnumerable<T>.Select()` and `IQueryable<T>.Select()` is that, even if your lambda is an expression lambda, the LINQ provider for your specific `IQueryable<T>` may not be able to translate the expression into operations supported by the underlying data source.  As an example, consider our earlier very simple example of `(x, i) => $"{i}: {x}"`.  This is an expression lambda with an interpolated string, which the compiler will compile as a method call expression which calls `string.Format()`.  A LINQ database provider which expects to be able to convert all its expression trees into SQL queries may well throw up its hands and say that a method call cannot be converted.  Even simple projection expressions such as `x => new { x.Width, x.Colour }` may not always work with all providers.  It is very easy, when using more advanced LINQ use-cases, to come up against spots like this where code that works on paper, or that works when unit tested with a mock `IQueryable<T>`, does not work when fully integrated with its intended data sources.  Moreover, these issues will often only be found at run-time.  One of the flaws of LINQ is that it obscures boundaries between providers, and thereby boundaries regarding functionality or locality of code.

#### More of the LINQ API

The API that all LINQ providers should implement is called the Standard Query Operator API.  We've already discussed `Select()`, as probably the most frequently-used methods, but here are the others.  The method signatures I've given here are the `IEnumerable<T>` version that take `Func<>` delegates, not the versions that take expression trees, but they are essentially the same in outline.

```
public static IEnumerable<S> Where<S>(this IEnumerable<S> source, Func<S, bool> predicate);
public static IEnumerable<S> Where<S>(this IEnumerable<S> source, Func<S, int, bool> predicate);

IEnumerable<Egg> eggs = GetSomeEggs();
IEnumerable<Egg> wholeEggs = eggs.Where(e => !e.Cracked);

public static IEnumerable<T> OfType<T>(this IEnumerable source);

IEnumerable<ChocolateEgg> = eggs.OfType<ChocolateEgg>();
```

The `Where()` method provides filtering.  It expects a predicate lambda that returns `bool`; the output sequence contains the elements for which the predicate returns `true`.  Like `Select()`, there is a form which takes the index of each element as a second parameter to the predicate.

The `OfType()` method is similar to `Where(x => x is T)` but also implies a cast to the given type, so it's effectively equivalent to `Where(x => x is T).Select(x => (T)x)`.  It is unusual, in that it always needs the type parameter to be specified, which does prevent you selecting the type in question dynamically.

```
public static T First<T>(this IEnumerable<T> source);
public static T FirstOrDefault<T>(this IEnumerable<T> source);
public static T First<T>(this IEnumerable<T> source, Func<T, bool> predicate);
public static T FirstOrDefault<T>(this IEnumerable<T> source, Func<T, bool> predicate);

public static T Last<T>(this IEnumerable<T> source);
public static T LastOrDefault<T>(this IEnumerable<T> source);
public static T Last<T>(this IEnumerable<T> source, Func<T, bool> predicate);
public static T LastOrDefault<T>(this IEnumerable<T> source, Func<T, bool> predicate);

public static T Single<T>(this IEnumerable<T> source);
public static T SingleOrDefault<T>(this IEnumerable<T> source);
public static T Single<T>(this IEnumerable<T> source, Func<T, bool> predicate);
public static T SingleOrDefault<T>(this IEnumerable<T> source, Func<T, bool> predicate);
```

This family of twelve methods all extract a single element from a list according to various criteria.  The `...OrDefault()` forms all return `default(T)` if no suitable element is found to return; the forms without `...OrDefault()` throw an `InvalidOperationException` if there is no suitable element.  All overloads of `Single()` and `SingleOrDefault()` also throw an `InvalidOperationException` if multiple elements match the predicate criteria.

The one-parameter versions of the methods are largely self-explanatory.  `First()` returns the first element, throwing an exception if the sequence is empty.  `Last()`, unsurprisingly, returns the last, again throwing an exception on an empty sequence.  `Single()` also returns the first element, the difference with `First()` being that it throws an exception unless the sequence *only* contains a single element.  `SingleOrDefault()` throws an exception if the sequence contains multiple elements, but not if it is empty.  If you are dealing with a sequence whose elements have a guaranteed order, `First()` and  `Last()` can straightforwardly be replaced with equivalent indexer operations: `[0]` and `[^1]`.

The two-parameter versions of the methods all require a predicate to narrow down the selection criteria.  In other words, `First()` will return the first element for which the predicate is true (and throw an exception if there are none), `Last()` will return the last for which the predicate is true (or throw an exception), and `Single()` will apply the predicate to all elements, throw an exception if none or more than one match, and return the single matching element otherwise.

```
public static T ElementAt<T>(this IEnumerable<T> source, int index);
public static T ElementAtOrDefault<T>(this IEnumerable<T> source, int index);
public static bool Contains<T>(this IEnumerable<T> source, T value);
public static bool Contains<T>(this IEnumerable<T> source, T value, IEqualityComparer<T> comparer);
public static int Count<T>(this IEnumerable<T> source);
public static int Count<T>(this IEnumerable<T> source, Func<T, bool> predicate);
```

These methods all largely provide support for operations that already exist in fixed-index collections.  `array.ElementAt(x)` is the equivalent of `array[x]`, and `ElementAtOrDefault()` provides a form of bounds-checking safety, although if it does return the default value there is no way to tell whether the index was actually in or out of bounds.  The `Count()` method also has an overload that takes a predicate, to determine how many elements in the sequence match the predicate.

```
public static IEnumerable<T> Take<T>(this IEnumerable<T> source, int count);
public static IEnumerable<T> TakeWhile<T>(this IEnumerable<T> source, Func<T, bool> predicate);
public static IEnumerable<T> TakeWhile<T>(this IEnumerable<T> source, Func<T, int, bool> predicate);
public static IEnumerable<T> Skip<T>(this IEnumerable<T> source, int count);
public static IEnumerable<T> SkipWhile<T>(this IEnumerable<T> source, Func<T, bool> predicate);
public static IEnumerable<T> SkipWhile<T>(this IEnumerable<T> source, Func<T, int, bool> predicate);
```

These methods both return a sequence consisting of a contiguous subset of the original sequence, in the same order.  `Take(n)` returns the first `n` elements from the source sequence and discards the rest; `Skip(n)` discards the first `n` elements and returns the rest.  Together, they can be used to extract arbitrary slices.  `TakeWhile()` returns a contiguous set of elements from the start of the sequence that match the given predicate, and discards the first element that does not match the predicate and all elements afterwards.  `SkipWhile()`, as you might expect, discards a contiguous set of elements from the start of the sequence that match the predicate, and returns the first element that does not match and all elements after it.  Both `TakeWhile()` and `SkipWhile()` also accept predicates with an index parameter.

`x.Take(n)` is the equivalent of the range-based indexer operation `x[..n]`, and `x.Skip(n)` is the equivalent of the range-based indexer operation `x[n..]` on types that guarantee ordering of their elements.

For the next group of methods, I have shown `int`-based signatures, but there are actually overloads for every numeric type.

```
public static int Sum(this IEnumerable<int> source);
public static int Sum<T>(this IEnumerable<T> source, Func<T, int> selector);

public static int Min(this IEnumerable<int> source);
public static int Min<T>(this IEnumerable<T> source, Func<T, int> selector);

public static int Max(this IEnumerable<int> source);
public static int Max<T>(this IEnumerable<T> source, Func<T, int> selector);

public static double Average(this IEnumerable<int> source);
public static double Average<T>(this IEnumerable<T> source, Func<T, int> selector);
```

These methods carry out basic numeric aggregate operations, either on a sequence of a numeric type, or on a sequence of any type when provided with a projection method to give the value to be used for the computation for each element.  Note that the result of the `Average(IEnumerable<int>)` method is a `double`; all of the `Average()` overloads return `double` except for `Average(IEnumerable<float>)` which returns `float`.

```
public static T Aggregate<T>(this IEnumerable<T> source, Func<T, T, T> func);
public static R Aggregate<S,R>(this IEnumerable<S> source, R seed, Func<R,S,R> func);
public static R Aggregate<S,A,R>(this IEnumerable<S> source, A seed, Func<A,S,A> func, Func<A,R> selector);
```

The `Aggregate()` methods are generalisations of routines such as `Sum()` which produce a single result by applying a method to successive elements in a sequence.  The first variant, with a single type parameter, must return a value of the same type as the elements, and also has to use the same type to store its intermediate values.  If it is passed a sequence with one value then that value is returned.  With a sequence of two values, the two values are passed as parameters to the lambda and the result returned.  With longer sequences, the first two values are passed to the lambda, then the result is passed to the lambda with the third value, then the result of that passed to the lambda with the fourth value, and so on until after the whole sequence has been passed to the lambda together with the result of the previous call, the result of the final invocation is returned as the overall result of the `Aggregate()` call.  This all sounds complex to explain, but can easily implement routines like, say, the `Sum()` method mentioned previously:

```
List<int> values = LoadData();
int sum = values.Aggregate(static (a, b) => a + b);
```

In the two-type-parameter version, the return type of the aggregate is different to the type of the source elements, but is the same type used for storing the intermediate value of the computation.  With this version, we have to provide an initial "seed" value for the intermediate value, and the lambda is applied to every element in the sequence in the same way, rather than the first two elements in the sequence being a special case.

The three-type-parameter version is similar to the above, but allows the return type to be a different type to the intermediate value.  An additional lambda method is provided to transform the final intermediate value into the return value of the method.  We can use this version of `Aggregate()` to implement the `Average()` method described above, using an anonymous type to store the intermediate value:

```
public static double Average(this IEnumerable<int> source)
{
    return source.Aggregate(
        new { Total = 0, Items = 0 },
        (a, v) => new { Total = a.Total + v, Items = a.Items + 1 },
        a => a.Total / (double)a.Items
    );
}
```

I've put the three positional parameters on separate lines for clarity: the first creates our intermediate seed as an instance of an anonymous class suitable for tallying a running total and element count, the second creates a new intermediate value by adding on the properties of the current element to the previous intermediate value, and the third outputs the final result by dividing the running total by the count of elements.  Note that in the real world, although using an anonymous class like this is nicely concise, it's not the most efficient way of going about things as you are briefly creating and then throwing away a lot of small immutable heap-allocated objects; a more efficient way would be to define a nested helper struct that is allocated once and mutated.  (In .NET Core, incidentally, `Average()` isn't built on top of `Aggregate()` at all, but [implemented in its own right.](https://github.com/dotnet/corefx/blob/master/src/System.Linq/src/System/Linq/Average.cs))

```
public static bool Any<T>(this IEnumerable<T> source);
public static bool Any<T>(this IEnumerable<T> source, Func<T, bool> predicate);
public static bool All<T>(this IEnumerable<T> source, Func<T, bool> predicate);
```

The one-argument form of `Any()` returns `true` if the source parameter contains any elements, and returns `false` if it is empty.  The other form of `Any()`, and `All()`, are analogous to the `||` and `&&` operators: the former returns `true` if the predicate returns `true` for any element, and the latter returns `true` if the predicate returns `true` for every element.  Like the `||` and `&&` operators, the methods are "lazy" and will only enumerate as much of the sequence as is necessary to determine the result.  In other words, `Any()` will return as soon as it finds an element for which the predicate is `true`, and `All()` will return as soon as it finds an element for which the predicate is `false`.

```
public static IOrderedEnumerable<T> OrderBy<T,K>(this IEnumerable<T> source, Func<T,K> keySelector);
public static IOrderedEnumerable<T> OrderBy<T,K>(this IEnumerable<T> source, Func<T,K> keySelector, IComparer<K> keyComparer);
public static IOrderedEnumerable<T> OrderByDescending<T,K>(this IEnumerable<T> source, Func<T,K> keySelector);
public static IOrderedEnumerable<T> OrderByDescending<T,K>(this IEnumerable<T> source, Func<T,K> keySelector, IComparer<K> keyComparer);
public static IOrderedEnumerable<T> ThenBy<T,K>(this IOrderedEnumerable<T> source, Func<T,K> keySelector);
public static IOrderedEnumerable<T> ThenBy<T,K>(this IOrderedEnumerable<T> source, Func<T,K> keySelector, IComparer<K> keyComparer);
public static IOrderedEnumerable<T> ThenByDescending<T,K>(this IOrderedEnumerable<T> source, Func<T,K> keySelector);
public static IOrderedEnumerable<T> ThenByDescending<T,K>(this IOrderedEnumerable<T> source, Func<T,K> keySelector, IComparer<K> keyComparer);
```

These methods are used for multi-level sorting. `OrderBy()` sorts elements in the list in ascending order,according to the value returned for each element by the `keySelector` method.  `OrderByDescending()`, naturally, sorts elements in reverse order.

The `ThenBy()` methods are used for secondary sorting.  The first parameter to `ThenBy()` is an `IOrderedEnumerable<T>` as returned by `OrderBy()` or `ThenBy()`; the `ThenBy()` method will only sort groups of adjacent elements in the ordered enumerable which compared equal in the previous sort.  The idea is that your code can look something like this:

```
var IEnumerable<Egg> eggs = LoadSomeData();
var sortedEggs = eggs.OrderBy(e => e.Diameter).ThenBy(e => e.Length).ThenByDescending(e => e.Colour);
```

This should then produce relatively readable code, the intent of which should be fairly obvious (notwithstanding how you might actually order something by colour).  Note that you can provide a specific `IComparer<T>` implementation to define exactly how the key values should be sorted.

```
public static IEnumerable<T> Reverse<T>(this IEnumerable<T> source);
```

Another ordering method, `Reverse()` is much more straightforward and does exactly what the name suggests: returns a sequence containing the same elements in reversed order.

```
public static IEnumerable<T> Concat(this IEnumerable<T> first, IEnumerable<T> second);
```

Another straightforward method, `Concat()` appends the second parameter on to the first.  Here's an example:

```
int[] first = new[] { 1, 2, 3 };
List<int> second = new List<int>() { 1, 3, 9, 2 };
var result = first.Concat(second);
// result's elements are 1, 2, 3, 1, 3, 9, 2
```

The two sequences do have to be of the same type.

```
public static IEnumerable<R> SelectMany<S, R>(this IEnumerable<S> source, Func<S, IEnumerable<R>> selector);
public static IEnumerable<R> SelectMany<S, R>(this IEnumerable<S> source, Func<S, int, IEnumerable<R>> selector);
public static IEnumerable<R> SelectMany<S, C, R>(this IEnumerable<S> source, Func<S, IEnumerable<C>> collectionSelector, Func<S, C, R> resultSelector);
public static IEnumerable<R> SelectMany<S, C, R>(this IEnumerable<S> source, Func<S, int, IEnumerable<C>> collectionSelector, Func<S, C, R> resultSelector);
```

The `SelectMany()` method is, like `Select()`, a form of projection operation.  In its two-type-parameter forms it uses `selector` to project each element of the sequence into a new sequence, then concatenates those sequences together to form the final output.  Here's an example:

```
// assume there is a class, Vertex
public class Polygon
{
    public Vertex[] Vertexes;
}

List<Polygon> polygons = CreatePolygons();
IEnumerable<Vertex> allVertexes = polygons.SelectMany(p => p.Vertexes);
```

Here, the selector method takes a `Polygon` instance and projects it to an `IEnumerable<Vertex>`, and the end result will be the concatenation of all `Vertexes` of all polygon elements.

With the three-type-parameter form, the source elements are each projected into an enumeration of a third type, and a second "result projection" method projects the elements of that type into the final result elements.  The result projection method takes two parameters, the first being an element from the source sequence and the second being one of the elements of the sequence that the first was projected into.

With both of the above types of `SelectMany()`, there are overloads which also pass the source element index to the first projection method, much as there is for other LINQ methods.

```
public static bool SequenceEqual<T>(this IEnumerable<T> first, IEnumerable<T> second);
public static bool SequenceEqual<T>(this IEnumerable<T> first, IEnumerable<T> second, IEqualityComparer<T> comparer);
```

The `SequenceEqual()` method returns true if two sequences contain the same number of elements, and the elements at each position are equal.  The first overload uses the default equality comparer for the type, so the exact behaviour of the method will depend on the type and whether it implements the `IEquatable<T>` interface or overrides the `object.Equals()` method.  If it does not, and the type is a reference type, this method will only return true if the elements at each position in the sequence are the same individual instances of objects because it will compare the values of their references.  The second overload allows you to provide a custom comparer, to control exactly how the elements are compared.

```
public static IEnumerable<T> Distinct<T>(this IEnumerable<T> source);
public static IEnumerable<T> Distinct<T>(this IEnumerable<T> source, IEqualityComparer<T> comparer);
```

The `Distinct()` method returns the elements in the sequence, with any duplicates removed.  For the first overload the same caveats, as to what counts as a duplicate, apply as for the `SequenceEqual()` method: essentially, if the type does not define custom equatability code, objects will only be considered duplicate if they are the same instances.  Again, there is a second overload that allows a custom comparer to be provided.  You should not assume the order of elements output by this method; providers are not required to guarantee that the output will be in any specific order, or that the order will be related to the order of the source sequence in any particular way.

```
public static IEnumerable<T> Union<T>(this IEnumerable<T> first, IEnumerable<T> second);
public static IEnumerable<T> Union<T>(this IEnumerable<T> first, IEnumerable<T> second, IEqualityComparer<T> comparer);
public static IEnumerable<T> Intersect<T>(this IEnumerable<T> first, IEnumerable<T> second);
public static IEnumerable<T> Intersect<T>(this IEnumerable<T> first, IEnumerable<T> second, IEqualityComparer<T> comparer);
public static IEnumerable<T> Except<T>(this IEnumerable<T> first, IEnumerable<T> second);
public static IEnumerable<T> Except<T>(this IEnumerable<T> first, IEnumerable<T> second, IEqualityComparer<T> comparer);
```

This group of methods carries out set operations on the two sequences: the set union, set intersection and set difference.  For `Union()` and `Intersect()` the operation is commutative; for `Except()` it is not.  All of these methods only return unique members, so for example `a.Union(b)` should be functionally equivalent to `a.Distinct().Union(b.Distinct())`.  Again, you should not make any assumptions about the order of the output sequence.

```
public static IEnumerable<IGrouping<K,S>> GroupBy<S,K>(this IEnumerable<S>, Func<S,K> keySelector);
public static IEnumerable<IGrouping<K,S>> GroupBy<S,K>(this IEnumerable<S>, Func<S,K> keySelector, IEqualityComparer<K> comparer);
public static IEnumerable<IGrouping<K,E>> GroupBy<S,K,E>(this IEnumerable<S> source, Func<S,K> keySelector, Func<S,E> elementSelector);
public static IEnumerable<IGrouping<K,E>> GroupBy<S,K,E>(this IEnumerable<S> source, Func<S,K> keySelector, Func<S,E> elementSelector, IEqualityComparer<K> comparer);
public static IEnumerable<R> GroupBy<S,K,R>(this IEnumerable<S> source, Func<S,K> keySelector, Func<K,IEnumerable<S>,R> resultSelector);
public static IEnumerable<R> GroupBy<S,K,R>(this IEnumerable<S> source, Func<S,K> keySelector, Func<K,IEnumerable<S>,R> resultSelector, IEqualityComparer<K> comparer);
public static IEnumerable<R> GroupBy<S,K,E,R>(this IEnumerable<S> source, Func<S,K> keySelector, Func<S,E> elementSelector, Func<IEnumerable<E>,R> resultSelector);
public static IEnumerable<R> GroupBy<S,K,E,R>(this IEnumerable<S> source, Func<S,K> keySelector, Func<S,R> elementSelector, Func<IEnumerable<R>,R> resultSelector, IEqualityComparer<K> comparer);
```

Now we are getting down towards the more complex methods in LINQ, for grouping and (below) joining.  The `GroupBy()` method overloads handle grouping of a sequence.  Here, I have tried to order them from simplest to most complex, and I've used consistent naming for the type parameters: S for source type, K for key type, E for element type and R for result type.  Each overload is paired with a second which includes a key equality comparer, but I'll gloss over those initially.

The simplest versions of `GroupBy()` return `IEnumerable<IGrouping<K,S>>`.  An `IGrouping<K,S>` is an extension of `IEnumerable<S>` which also includes a key value; in other words, these methods return a sequence of sequences, where each "interior" sequence is identified by a key.  The two-parameter overload takes a method which will project each element to a key, then returns the elements grouped by that key value.  In other words, the following code:

```
IEnumerable<Egg> eggs = LoadData();
IEnumerable<IGrouping<string,Egg>> filedByColour = eggs.GroupBy(e => e.Colour); // assume Colour is a string
```

will take the initial sequence of `Egg` objects and return a sequence of sequences, with each second-level sequence having the same `Colour` property.  The next overloads, which are of the loose form `GroupBy(source, keySelector, elementSelector)` work in the same way, but include an element selector method to project each element into a new form: it is similar to calling the first form of the method, but carrying out a `Select()` operation on each element as it's put into its group.

The next forms of the method have the loose form `GroupBy(source, keySelector, resultSelector)`.  Unlike the previous form the result selector here operates on the output groups, not their elements, and has the type `Func<K,IEnumerable<S>,R>`.  Its parameters are the group key and the sequence of elements in the group.  Instead of each group being output as an `IGrouping`, each combination of key and subsequence can be turned into a different type of object, so you can use this form to return aggregate-level data on a group rather than the group itself.

The final forms combine the previous two.  The key selector method derives the key for each element; and the element selector projects each element into an intermediate form.  The elements are grouped, and the result selector is called for each group to create the final output elements.

If a comparer parameter is provided, it is used to compare the values of the group keys when grouping the elements of the input sequence, to determine whether an element belongs to an existing group, or is in its own group.

```
public static IEnumerable<R> Join<U,N,K,R>(this IEnumerable<U> outer, IEnumerable<N> inner, Func<U,K> outerKeySelector, Func<N,K> innerKeySelector, Func<U,N,R> resultSelector);
public static IEnumerable<R> Join<U,N,K,R>(this IEnumerable<U> outer, IEnumerable<N> inner, Func<U,K> outerKeySelector, Func<N,K> innerKeySelector, Func<U,N,R> resultSelector, IEqualityComparer<K> comparer);
```

The `Join()` method is the equivalent of the SQL `INNER JOIN` statement.  It takes an "outer" sequence, an "inner" sequence", a key selector method for each sequence, and a result selector method.

The operation of the method is to iterate over both sequences and apply the relevant key selector to each element of each sequence.  Then, for each key in the outer sequence, elements with matching keys are found in the inner sequence, and result selector method is called for each of these pairs, to generate an output element.  As for `GroupBy()`, the version with the comparer parameter uses that comparer to compare the derived keys for equality.

Unlike some of the other LINQ methods, `Join()` preserves the order of its input.  The overall order of the output elements should match the order of the outer sequence, and within groups of output elements generated from the same outer element, the output order should match the order of the inner sequence.

As this is the equivalent of an SQL join, it might be worth comparing it with an equivalent SQL statement.  Imagine we have two tables, named Gender and Name, with the following data:

| GenderID | DisplayValue |
| -------- | ------------ |
| 0        | Other        |
| 1        | F            |
| 2        | M            |
| 3        | NB           |
| 4        | Not stated   |

| NameID | Name     | GenderID |
| ------ | -------- | -------- |
| 0      | Caitlin  | 1        |
| 1      | Dave     | 2        |
| 2      | Theo     | 3        |
| 3      | Gretchen | 1        |
| 4      | Lavinia  | 1        |
| 5      | Alex     | 0        |
| 6      | Ezra     | 2        |

We can combine these tables using a SQL statement like: 

```
SELECT n.Name, g.DisplayValue AS Gender 
FROM Name n 
  INNER JOIN Gender g ON n.GenderID = g.GenderID 
ORDER BY n.NameID, g.GenderID
```  

This would produce the following output:

| Name     | Gender |
| -------- | ------ |
| Caitlin  | F      |
| Dave     | M      |
| Theo     | NB     |
| Gretchen | F      |
| Lavinia  | F      |
| Alex     | Other  |
| Ezra     | M      |

The LINQ code to do this would be as follows:

```
// Assume IEnumerable<Name> names and IEnumerable<Gender> genders
return names.Join(genders, n => n.GenderID, g => g.GenderID, (n, g) => new { n.Name, Gender = g.DisplayValue });
```

Note the parts that are equivalent to the SQL, and the parts that cannot be changed.  The `ORDER BY` clause we specified in the SQL is effectively fixed in the LINQ code; to order it differently, we would have to pass the output to an `OrderBy()` call.  The two sides of the `ON` clause in the SQL are changed into the two selector methods that select the `GenderID` property, but the equality operator in the `ON` clause is hard-coded into the LINQ method.  The final method in the LINQ call, the result selector method, is effectively the equivalent of the output column list in the `SELECT` statement.

The final method we are going to look at in this section, `GroupJoin()`, is a more flexible form of `Join()` that is similar to the SQL `LEFT [OUTER] JOIN` statement.

```
public static IEnumerable<R> GroupJoin<U,N,K,R>(this IEnumerable<U> outer, IEnumerable<N> inner, Func<U,K> outerKeySelector, Func<N,K> innerKeySelector, Func<U,IEnumerable<N><R>> resultSelector);
public static IEnumerable<R> GroupJoin<U,N,K,R>(this IEnumerable<U> outer, IEnumerable<N> inner, Func<U,K> outerKeySelector, Func<N,K> innerKeySelector, Func<U,IEnumerable<N><R>> resultSelector, IEqualityComparer<K> comparer);
```

The difference between the `Join()` and `GroupJoin()` methods is that whereas the result selector method of `Join()` is called once per matching pair of elements, the result selector method of `GroupJoin()` is called once and once only for *every* outer element.  The first parameter to the result selector is the outer element; the second parameter is the sequence of inner elements that match that outer element, which may be an empty sequence.  This means that `GroupJoin()` can be used to implement either an inner or an outer join when used in combination with other LINQ methods.  To have it behave directly like a SQL `JOIN` you will generally have to pass the output of `GroupJoin()` to `SelectMany()` to flatten the results down to a single-dimension sequence; in a lot of contexts, though, its default behaviour is more flexible and useful.

#### The other LINQ syntax

The special LINQ syntax is called "query syntax".  Officially, Microsoft recommend that you "use query syntax whenever possible and method syntax whenever necessary".  However, in reality, the majority of C# developers (that I've met) probably prefer to use method syntax and generally do not use query syntax.  The output of an expression in query syntax is exactly the same as for method syntax (some sort of `IEnumerable<T>` implementation that encapsulates the query ready for execution when enumerated), and query syntax compiles to exactly the same CIL method calls as method syntax, so it really comes down to an issue of personal taste and your opinions on readability.  Microsoft's official opinion is that query syntax is more readable; developers who are less confident using it than method syntax would understandably disagree.

The first example we gave for the `Select()` method above was this:

```
double[] values = new[] { 0.6, 7.2, 138.94, -8642.8 };
var output = values.Select(x => x * 2);
```

Using query syntax this would be the following:

```
double[] values = new[] { 0.6, 7.2, 138.94, -8642.8 };
var output = from x in values select x * 2;
```

If you are familiar with SQL you can see how this looks a bit like a SQL query swapped back to front.  Like in SQL the parts of a query expression are called *clauses*.  Each expression must start with a `from` clause and end with either a `select` or `group` clause.  In between there can be optional filtering and ordering clauses:

```
var output = from x in values
    where x > 0
    orderby x
    select x * 2;
```

This is the equivalent to:

```
var output = values.Where(x => x > 0).OrderBy(x => x).Select(x => x * 2);
```

With a query expression, though, if you want to put the clauses in a different order you have to break it up into separate parts.  If your method syntax expression is this:

```
var output = values.Select(x => (x + 1) * 2).Where(x => x > 0).OrderBy(x => x);
```

then to turn it into a query expression, you have to split it into two parts, as follows:

```
var intermediate = from x in values 
    select (x + 1) * 2;
var output = from x in intermediate
    where x > 0
    orderby x
    select x;
```

You can nest your expressions in parentheses, but it doesn't really help for readability, in my opinion

```
var output = from x in (from y in values select (y + 1) * 2)
    where x > 0
    orderby x
    select x;
```

If you really want to, you can mix the two types of syntax together:

```
var output = (from y in values select (y + 1) * 2).Where(x => x > 0).OrderBy(x => x);
```

In my opinion this is probably a bad idea unless you want to try to confuse people.

You can also include a join clause in a query expression.  The equivalent of our example from above:

```
// Assume IEnumerable<Name> names and IEnumerable<Gender> genders
return names.Join(genders, n => n.GenderID, g => g.GenderID, (n, g) => new { n.Name, Gender = g.DisplayValue });
```

would be something like:

```
return from n in names
    join g in genders on n.GenderID equals g.GenderID
    select new { n.Name, Gender = g.DisplayValue };
```

Note the use of the `equals` keyword to separate the two key selector expression methods, indicating that you can't substitute another operator in there.

As shown above, the query expression syntax is rather more restrictive than method syntax, both in terms of the order of the clauses, but also in the expressivity of each clause.  Personally I would always use the method syntax, but hopefully reading this you have enough information to recognise query syntax when you see it, and understand the meaning of its clauses.

### String handling

String handling is a basic function of almost all higher-level programming languages, so it's worth you knowing how to carry out some basic string operations in C#.

Strings are a fundamental type in .NET, although they behave as if they are a kind of object.  Indeed, in CIL itself "object" and "string" are given as the two fundamental reference data types, but with the rider that "string" instances must always behave as if they inherit from "object" despite being a different fundamental type at the bytecode.  In C# `string` appears to be defined as a sealed class which inherits from object, but this really just reflects the underlying implementation and is the best way to represent the CIL behaviour required by the CLR standards in C#.

A fundamental tenet of .NET strings is that they are always immutable.  All string operations and methods, therefore, will return a *different* instance to the one(s) being operated on, as you cannot mutate an existing string.  This naturally affects the style of the string API, but also is sometimes raised as a possible performance problem, because certain idioms such as constructing strings piece-by-piece or by repeated concatenation can cause a large number of short-lived strings to be created.

Whether this "performance problem" is actually a problem in the real world depends very much on what you are doing.  Because of a perception that it may be a problem, there is a class, `System.Text.StringBuilder`, intended to be used in situations where you may want to build up a string in multiple steps.  It effectively behaves like a kind of mutable string buffer, with limited replacement ability; its contents can then be turned into a regular .NET string by calling `StringBuilder.ToString()`.  Because there is an overhead implied by using `StringBuilder`, the point at which use of `StringBuilder` becomes more efficient than direct string operations is very hard to predict and can vary hugely based on the exact algorithm used, the CLR implementation, the amount of data involved, and anything else which can affect the behaviour of the garbage collector.

The `string` class implements `IEnumerable<char>`, so if you find that the available string API does not easily lead to the operation required, you can use LINQ methods to filter and transform your string as a sequence of characters and turn it back into a string afterwards.  Annoyingly, there is no `string(IEnumerable<char>)` constructor; however, there is a constructor that takes `char[]`, so you can get around this with the `IEnumerable<char>.ToArray()` method.

The `string` class implements an `int`-parameter indexer of type `char`.  As each instance is immutable, it's a read-only indexer, but can easily be used to access the character at a given position in the string.  Since C# 8 there is also an `Index`-parameter indexer and a range-based indexer of type `string`, which covers various situations that previously required string manipulation methods.  For example, the expression `myString[x..]` is equivalent to `myString.Substring(x)`.

C# compiler implementations will generally de-duplicate string constants and literals in output assemblies.  This is done through a mechanism called the *intern pool*, which can be accessed at runtime, although doing so is not necessarily helpful.  Although there is no automatic deduplication of runtime-generated strings, it's not really productive to check to see if your string exists in the intern pool already: you can't check if it exists without constructing it anyway, and putting it into the pool will prevent it from being garbage-collected at all during the lifetime of the process.

The `string` class has a predefined static field, `Empty`, to refer to the empty string.  You may often see references to `string.Empty` (or `String.Empty`) in code instead of the equivalent `""`.  In C# 1.0, using `string.Empty` was felt by a lot of developers to convey a minor performance improvement over `""` because the earliest compiler didn't have string constant deduplication; however, any such improvement was likely to be minimal in practice, and the two are essentially synonyms.  A large number of developers still prefer to use `string.Empty` for readability or personal preference, but it is functionally equivalent to `""`, and a modern CLR implementation should use a singleton instance as its empty string.  If you are comparing a value against an empty string, it is better practice to consider how you want to handle null values and white space values too and use `string.IsNullOrEmpty(val)` or `string.IsNullOrWhitespace(val)` if they give the semantics you need without requiring an additional null check.

#### Composite format strings

We have already discussed above the various types of string literal, and I mentioned that interpolated string literals of the form `$"..."` are actually syntactic sugar for calls to the `string.Format()` method.  This method has the following signature, in its most commonly-used form:

```
public static string Format(string format, params object[] args);
```

The `format` parameter is referred to as a *composite format string*, and differs somewhat from an interpolated string literal.  Whereas an interpolated literal contains expressions within `{}` pairs, a composite format string contains indexed placeholders, with their indexes referring to elements of the `args` array.  When discussing string literals we used the example `$"There are {lines.Count} lines in the document.";`.  As a `Format()` call, this turns into `string.Format("There are {0} lines in the document.", lines.Count);`.  Note how the expression is moved into the `args` array, and within the format string it is replaced by a placeholder; as there is only one member of the array, its placeholder is the first index value `{0}`. 

(To be very pedantic, because there are so many calls to `string.Format()` with a single argument following the format string, there is actually in this case a specific overload, `string.Format(string format, object arg0)`,  called to avoid the work involved in constructing an array and counting the number of parameters, but in operation it is functionally identical to the version given above).

If there are multiple placeholders in the format string, they do not have to occur in numerical order, but it can be somewhat confusing if they don't.  Of course, if you are writing internationalised code with multiple translations of your strings, it can be impossible for the placeholders to be in the same order in every translation of some strings.  You can also reuse the same placeholder multiple times, as in `"There are {0} lines in the document, yes, {0} lines!"`.  This can come in useful if you want to format the same value in different ways, for example, use both the date part and the time part of a `DateTime` in different parts of your format string.

You can specify the minimum width of the interpolated string that replaces a placeholder by following the placeholder with a comma and an integer; this is called the *alignment component*.  If the alignment is positive and greater than the "natural" width of the value, spaces are added before the value to right-align it; if the alignment is negative, spaces are added after the value to left-align it.

If the placeholder and any alignment component are followed by a colon and a further string, that further part is used to control exactly how the value is converted to a string.  The format strings used for this part are discussed in more detail below, and are the same as the format strings which can be passed to the `ToString()` method of a number of built-in types to control formatting.  Here's an example of a `string.Format()` call which uses all of the features we have talked about:

```
string timeString = string.Format("The time now is:\nHours: {0,5:H}\nMinutes: {0,3:m}\nSeconds: {0,3:s}\nMillis: {0,4:fff}", DateTime.Now);
```

This should generate a string that looks something like the following:

```
The time now is:
Hours:    16
Minutes:  35
Seconds:   3
Millis:  326
```

Incidentally, if you want to include a literal `{` or `}` character in your format string, you double it up.  However, the brace-escaping mechanism is not particularly intelligent and often does not properly handle cases where a placeholder is wrapped by braces.  This is a known issue; the recommendation, if you are trying to include literal braces and are getting strange results, is to pass them in via placeholders themselves.  Instead of writing `string.Format("{{{0:x}}}", data)`, write `string.Format("{0}{1:x}{2}", "{", data, "}")`.

There are other methods which accept composite format strings which follow the same standards as `string.Format()`.  For example, some overloads of the `Console.WriteLine()` and `TextWriter.WriteLine()` methods, which are used for writing strings to the console or generally to streams, support the use of format strings, and the  `StringBuilder` class mentioned above has `AppendFormat()`, for adding formatted text to its buffer.  Some external frameworks also support composite format strings, usually by calling `string.Format()` internally; for example the NLog logging framework supports the use of composite format strings for all of its methods.  In general, it is worth investigating to see if the objects you are using support passing composite format strings so that you can delegate that work to the library or framework code, rather than calling `string.Format()` yourself.  It can in some circumstances be more efficient.  For example, the NLog framework has various complex rules that can be set to determine where a given message should be logged to, or if it should be thrown away without being recorded.  In the latter case, the library can then avoid formatting the output string if it knows it is not going to use the output.  If you called `string.Format()` yourself before passing the result to the logger, you may be making a wasted call, but it would be very hard for you to be able to perform the optimisation yourself.

Interpolated string literals of the form `$"..."` use exactly the same placeholder syntax as `string.Format()`, with the big difference that the first part of the place holder, instead of being just an index number, is an entire expression that is evaluated and converted into a string.  This can potentially cause even more issues for the parser, if it's combined with alignment and format information.  As mentioned above in the section on string interpolation, the parser will get confused if you use the `? :` operator in your expression without putting parentheses around it, because it will assume the colon marks the start of the format specifier.  Additionally, your alignment and format specifiers have to be constants.  If you want to set your alignment programmatically, I usually find the easiest way is to use an interpolated string as the format parameter to a `string.Format()` call.

#### The main string API

We have already mentioned that strings implement `IEnumerable<char>` and are indexed by integer, by `System.Index` and by `System.Range`.  They also, like arrays, have an integer `Length` property.  The most commonly-used methods in the `string` class are probably the following:

- `Concat()` concatenates an array of strings into one.
- `CopyTo()` copies a slice of a string into a `char[]`.
- `EndsWith()` determines if a string ends with a specific substring, and `StartsWith()` determines if it starts with a specific substring.
- `IndexOf()` determines the first point within the string where a specific character or substring occurs, if it does.  `LastIndexOf()` does the same, but starts looking from the end of the string.
- `IsNullOrEmpty()` is static, and determines if a `string` reference is either null or the empty string, as discussed above.  `IsNullOrWhiteSpace()` is similar but returns true for strings that only contains white space characters.
- `Join()` concatenates a sequence of strings, with a specified separator (which could be the empty string) between each element of the sequence.
- `Remove()` returns a string with a slice removed.
- `Replace()` returns a string with a given substring replaced by something else.
- `Split()` splits a string into an array of strings, based on the position of one or more delimiters.
- `Substring()` extracts a slice from a string.
- `ToUpper()` and `ToLower()` return a string which contains equivalent characters all converted into upper or lower case respectively.  This is a culture-sensitive operation (see below).
- `Trim()` removes leading and trailing whitespace, or instances of a specific character: for example `"__eg__".Trim('_');` returns `"eg"`.  `TrimStart()` and `TrimEnd()` do the same at just one end of the string.

#### String cultures and encodings

.NET has a built-in concept of the program's current locale, based usually on operating system settings.  This can have a large impact on string handling code.  Some string handling routines use the current locale by default, and this can result in bugs if code developed on one computer under the developer's locale is transferred to a computer set to a different locale&mdash;for example when you take code you have tested on your local machine and start running it on a cloud-hosted server.  In my experience it is most often an issue when converting dates or numbers to strings, but don't assume those are the only places affected.

To avoid locale-based problems, many string operations have overloads which take a `System.IFormatProvider` parameter, into which you can pass a `System.Globalization.CultureInfo` object to specify which locale is to be used for the specific operation.  This can be used to hard-code a specific geographical culture, or the special *invariant culture*.  The invariant culture is described as locale-agnostic but in reality has a strong bias towards US English practices; however it can be used to ensure consistent behaviour when consistency is more important than human-readability.  The Microsoft document on string handling best practice recommends always using overloads that allow you to specify the locale explicitly when it is possible to do so, even thought this may require you to do things such as use the `string.Equals()` method instead of the `==` operator, and turning on all code analysis rules will give you a warning when you could change your code to use a locale-aware overload.  To get the necessary `CultureInfo` object, you can use one of the following:

- the static property `CultureInfo.InvariantCulture`, which is always the invariant culture.
- the static property `CultureInfo.CurrentCulture`, which is the current culture for your process.  This normally starts out as the default culture of the user who started a process, but can be changed by a process as it runs.
- the static property `CultureInfo.CurrentUICulture`, which can potentially be different to `CurrentCulture`.  It is the property always used to select translations for UI purposes.
- as a last resort, you can manually construct a `CultureInfo` object by name, for example `new CultureInfo("cy-GB")` to get the British Welsh locale.

I said the current culture can be changed at runtime: you can set the value of `CultureInfo.CurrentCulture` to a different `CultureInfo` object. However, this property is thread-specific and non-persistent. Therefore doing so will only change the culture for the current thread until it terminates.  If you are not aware of this behaviour, this can cause strange problems further down the line, particularly if you are using asynchronous code.  There are circumstances in which code like this might be useful, but in general you are safer just using culture-aware overload methods.

Internally, .NET implementations should all use UTF-16 as their string encoding.  However, by default, stream input and output uses UTF-8, and the fact UTF-16 strings are used internally in memory is generally quite well-hidden.

String encodings themselves are accessed through the `System.Text.Encoding` class, which has a number of static properties to access commonly-used encodings.  The encodings accessible are:

| Encoding | Property                       | Notes                                                                                                  |
| -------- | ------------------------------ | ------------------------------------------------------------------------------------------------------ |
| US ASCII | `Encoding.ASCII`               | 7-bit support only                                                                                     |
| UTF-7    | `Encoding.UTF7`                | Not recommended unless you know why you need it.                                                       |
| UTF-8    | `Encoding.UTF8`                | The default encoding when not specified for many types, particularly `StreamReader` and `StreamWriter` |
| UTF-16   | `Encoding.Unicode`             | Either byte order supported.                                                                           |
| UTF-32   | `Encoding.UTF32`               | Either byte order supported.                                                                           |
| Other    | `Encoding.GetEncoding(int cp)` | Loads the encoding for any system-supported code page, by code page number.  This is operating-system specific; few code pages are guaranteed to be supported by all systems. |

If you want to ensure the widest variety of encodings are available, reference the NuGet package `System.Text.Encoding.CodePages`.  In modern versions of .NET, you are requireed to use this in order to access "code-page-based" encodings such as those used by Windows before it adopted Unicode, and those defined in the ISO 8859 series.

There is also a property, `Encoding.Default`, which theoretically specifies the default encoding for the system; however, the value of this is implementation-dependent.  On .NET and .NET Core it is guaranteed to always be `Encoding.UTF8`, but on .NET Framework running on Windows, it will be the system default code page encoding.

If you need to encode a string using a specific encoding, you can do it with `Encoding.GetBytes()` as follows:

```
string theString = "This is an example string.";
byte[] encodedString = Encoding.Unicode.GetBytes(theString);
Console.WriteLine($"UTF-16: {encodedString.Length} bytes.");
encodedString = Encoding.ASCII.GetBytes(theString);
Console.WriteLine($"ASCII: {encodedString.Length} bytes.");
```

As characters require two bytes to be encoded in UTF-16 but one byte for ASCII, this prints out the following:

```
UTF-16: 52 bytes.
ASCII: 26 bytes.
```

To go the other way, use the `Encoding.GetString()` or `Encoding.GetChars()` methods.

#### ToString() formats

All objects in C# support the `ToString()` method, as it is a member of `object`.  The `object.ToString()` method itself is not *particularly* useful, because it just returns the fully-qualified specific type name of the instance, but overrides and overloads of `ToString()` do provide useful functionality.  Many key built-in types implement `IFormattable`, which means they provide `ToString()` overloads which take a format string parameter, determining how exactly to carry out the conversion.  This interface is implemented by (among others) numeric types; dates, times and timespans; GUIDs and enums.  These format strings are the same as the format specifier component of placeholders in composite format string&mdash;in other words, the part between the colon and the closing brace.  Some format strings are locale-specific, and so the `IFormattable` interface also requires its implementers to provide a `ToString(string format, IFormatProvider provider)` overload so the caller can pass in a `CultureInfo` object.

For most of the types that support `ToString(string format, IFormatProvider provider)`, there are two types of format string: a "standard format string" consisting of one or two characters, and a more complex "custom format string".  The exceptions are the `Guid` type and enum types, which each only support a small number of standard format strings.

| Format string | GUID meaning                                            | Enum meaning                                       |
| ------------- | ------------------------------------------------------- | -------------------------------------------------- |
| `B`           | Digits separated by hyphens, surrounded by braces.      | N/A                                                |
| `D`           | Digits separated by hyphens, no surround.               | Decimal integer value, without leading zeros.      |
| `F`           | N/A                                                     | If the value is one or more enum values OR'd together, the names of those values separated by commas.  If is not, the integer value. |
| `G`           | N/A                                                     | The name of the value.  If the type has the `[Flags]` attribute this behaves like the `F` string.  If the value has no name, the integer value. |
| `N`           | Digits with no separation and no surround.              | N/A                                                |
| `P`           | Digits separated by hyphens, surrounded by parentheses. | N/A                                                |
| `X`           | Sequence of hexadecimal values, surrounded by braces    | The hexadecimal integer value, with leading zeros. |

The `X` format string for `Guid` values is hard to explain except by example.  If `N` would output a given GUID as `ddfe484b6a91457086a29d816dd394fc` then `X` would output it as `{0xddfe484b,0x6a91,0x4570,{0x86,0xa2,0x9d,0x81,0x6d,0xd3,0x94,0xfc}}`.

Numeric standard format strings are of the form `Xnn`, where `X` is a single letter, the *format specifier* and `nn` is an optional integer between 0 and 99 inclusive called the *precision specifier* which controls the number of digits in the result string.  If the precision specifier is not given, each format specifier has its own default.  The standard format specifiers are as follows:

| Format specifier | Name        | Notes |
| ---------------- | ----------- | ----- |
| `C`              | Currency    | Very locale-specific.  Uses the locale's currency symbol, thousands separator, decimal separator, default precision specifier and way to represent negative numbers.  Some locales represent negative currency values by enclosing them in parentheses. |
| `D`              | Decimal     | Only supported by integer types.  Optional negative sign.  No thousands separator.  The default precision specifier is the minimum number of digits required to display the number fully. |
| `E`              | Exponential | Default precision specifier 6.  Uses locale's decimal separator.  The case of the format specifier determines the case of the "e" in the output. |
| `F`              | Fixed point | No thousands separator.  Uses locale's decimal separator and default precision specifier. |
| `G`              | General     | Whichever of `F` or `E` would give the shortest result string.  The default precision specifier depends on the input type. |
| `N`              | Number      | Uses the locale's thousands separator, decimal separator and default precision specifier. |
| `P`              | Percent     | Multiplies the value by 100, and appends a `%`.  Uses the locale's decimal separator and default precision specifier. |
| `R`              | Round trip  | Should only be used for the `System.Numerics.BigInteger` type.  With this type, can round-trip to string and back to `BigInteger` without losing accuracy or precision. |
| `X`              | Hexadecimal | Only supported by integer types.  Negative values are treated as if their two's complement binary representation, in the shortest data type in which they will fit, is cast to an unsigned value; for example -1 becomes `FF`.  The case of the format specifier indicates the case to be used for digits A to F. |

In general the above standard numeric format specifiers are case-insensitive, aside from the effects noted for `E` and `X`.

Custom numeric format specifiers have individual characters for each digit to be potentially output in the result; `0` is used for a position where a digit must be output even if there is no significant digit to go there, and `#` is used for a digit position that can be omitted if there is no significant digit to fill it. `.` and `,` represent the decimal and thousands separators respectively, but are replaced by their locale-specific versions on output.  You can also specify three patterns separated by `;` characters, to provide separate formatting for positive, negative and zero values.  Custom numeric format specifiers are somewhat more flexible than the standard format specifiers, and are the only way to output per mille values, but are required somewhat less often, so I'm not going to describe them completely here.

The standard and custom `DateTime` and `Timespan` format strings are a little bit more involved, because many refer to extracting parts of the data, such as displaying only the date from a `DateTime`.  The standard format strings are single characters; the custom format strings are more complex, and unlike numeric format strings they tend to be case-sensitive.  The standard `DateTime` format strings are as follows:

| Format specifier | Name | Example | Notes |
| --- | --- | --- | --- |
| `d` | Short date | 22/10/2019 | Locale-specific ordering of components. |
| `D` | Long date | Tuesday, 22nd October 2019 | Locale-specific with words translated according to locale. 
| `f` | Full date, short time | Tuesday, 22nd October 2019 4:29 PM | Locale-specific with words translated according to locale.  Whether the time uses the 12- or 24-hour clock is also locale-specific. |
| `F` | Full date, long time | Tuesday, 22nd October 2019 4:29:16 PM | Locale-specific with words translated according to locale.  Whether the time uses the 12- or 24-hour clock is also locale-specific. |
| `g` | General (ie short) date, short time | 22/10/2019 4:29 PM | Locale-specific order of date components and locale-specific 12- or 24-hour clock. |
| `G` | General (ie short) date, long time | 22/10/2019 4:29:16 PM | Locale-specific order of date components and locale-specific 12- or 24-hour clock. |
| `M` or `m` | Month and day | September 22 | Locale-specific order of components with words translated. |
| `O` or `o` | Round-trip | 2019-10-22T16:29:16.0000000+01:00 | Locale-independent. ISO8601 compliant. |
| `R` or `r` | RFC1123 compliant | Tue, 22 Sep 2019 15:29:16 GMT | Locale-independent. The string `GMT` is always output regardless of the data, so you must ensure the data is converted to UTC or WET/GMT first. |
| `s` | Sortable | 2019-10-22T16:29:16 | Not locale-specific.  ISO8601 compliant.  Does not output any timezone information (unlike `O`). |
| `t` | Short time | 4:29 PM | Locale-specific 12- or 24-hour clock. |
| `T` | Long time | 4:29:16 PM | Locale-specific 12- or 24-hour clock. |
| `u` | Universal sortable | 2019-10-22T16:29:16Z | Locale-independent.  ISO8601 compliant.  Like `R`, the "Zulu" timezone information is actually hardcoded, so you must ensure the data is converted to UTC first. |
| `U` | Universal full date, long time | Tuesday, 22nd October 2019 3:29:16 PM | Locale-specific with words translated.  12- or 24-hour clock also locale specific.  Unlike other forms, the data is automatically converted to UTC if possible before output. |
| `Y` or `y` | Year and month | September, 2019 | Locale-specific with words translated. |

Note that, in general, .NET does not have particularly good or consistent support for time zone handling, as is shown by the slightly scatty and inconsistent way in which `DateTime.ToString()` behaves in this regard: some formats assuming that the data they receive is always UTC, others converting it to UTC automatically.  The `DateTimeOffset` type gives you slightly better support as it includes an offset from UTC in the structure, but if you do need to ensure your code always behaves properly in a time-zone-aware way, you may want to consider using an external library such as [NodaTime](https://nodatime.org/).

The `TimeSpan` type only has a few standard format strings.

| Format specifier | Name | Notes |
| --- | --- | --- |
| `c`, `t` or `T` | Constant format | Not locale-sensitive.  Only includes significant parts of the result. |
| `G` | General long format | Locale-sensitive.  Always includes all parts of the result, even if they are non-significant zeros. |
| `g` | General short format | Locale-sensitive.  Only includes significant parts of the result. |

The custom `DateTime` and `TimeSpan` format string specifiers generally each specify a particular part of the date and time, such as "year" or "minutes", and the characters are repeated to specify an increased number of digits or length of output; for example `yy` versus `yyyy` for two or four digit years, or `ddd` versus `dddd` for "Tue" versus "Tuesday".  The full range of specifiers for `DateTime` and `DateTimeOffset` is as follows:

| Format specifier | Name | Notes | 
| --- | --- | --- |
| `d`, `dd` | Number of day in the month | `dd` is zero-padded to two digits. |
| `ddd`, `dddd` | Name of day | Translated according to locale.  `ddd` is the standard abbreviated name. |
| `f` to `ffffff` | Fractions of a second | Resolution from tenths (`f`) to ten-millionths, all zero-padded to the maximum number of digits. |
| `F` to `FFFFFF` | Fractions of a second | Like `f`, but without zero-padding. |
| `g` | Era, ie A.D. | You can't use `g` on its own as it would be interpreted as a standard format specifier, but `%g` would work (see `%` below).  Any number of consecutive `g` characters behaves the same as one.  Translated according to locale. |
| `h`, `hh` | Hour (12-hour clock) | `hh` is zero-padded. |
| `H`, `HH` | Hour (24-hour clock) | `HH` is zero-padded. |
| `K` | Time zone infomation | For `DateTime` can either return "Z", the operating system timezone, or an empty string, depending on the `DateTime.Kind` property.  For `DateTimeOffset`, returns the same as `zzz`. |
| `m`, `mm` | Minute | `mm` is zero-padded |
| `M` to `MMMM` | Month | `M` and `MM` are numbers (with the second form zero-padded), `MMM` the abbreviated name, `MMMM` the full name.  The latter two are translated according to locale. |
| `s`, `ss` | Second | `ss` is zero-padded |
| `t`, `tt` | AM or PM | Translated according to locale.  `t` returns the first character alone, so `A` or `P` in English locales. |
| `y` to `yyyyy` | Year | `y` gives the final two digits of the year with any leading zero suppressed.  Other forms give the same number of (least-siginificant) digits as the number of letters. |
| `z` to `zzz` | Time zone offset | For `DateTime` returns the offset of the operating system timezone from UTC, regardless of the value passed in.  For `DateTimeInfo` it returns the offset from UTC; `z` gives hours, `zz` gives zero-padded hours and `zzz` gives hours and minutes. |
| `:` | Time separator | Locale-specific. |
| `/` | Date separator | Locale-specific. |
| `"..."` or `'...'` | String literals | Any unrecognised characters are taken to be a string literal anyway, but quoting allows you to use format specifier characters in literals. |
| `%` | Custom-format indicator | Specifies that the following character is a custom format specifier.  For use in situations where you have a single-character custom format which cannot be distinguished from a standard format, such as `g` or `h`, or to remove ambiguity with groups of repeated characters.  For example, with the date October 10th, the custom formt `M%MMM` gives the result `10Oct` in an English-language locale, whereas `MMMM` gives `October`. |
| `\` | Escape character | Escapes a single character.  `\h` is equivalent to `'h'`. |

The `TimeSpan` class supports the `f`, `F`, `h`, `m` and `s` custom specifiers in the same way as `DateTime`, including all lengths, and also supports `d` to mean the number of days in the time period, padded up to eight digits (`dddddddd`).  It also supports the special characters `%` and `\` in the same way as described above.
 
#### Regular expressions

### Preprocessor Directives

### Events

### Locking

### Tuples

### Asynchronous code

### Dynamic typing

### Overflow checking

### Unsafe code
