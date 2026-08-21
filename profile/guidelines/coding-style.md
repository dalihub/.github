# Dali C++ Coding Style

## Naming

The most important consistency rules are those that govern naming. The style of a name immediately informs us what sort of thing the named entity is: a type, a variable, a function, a constant, a macro, etc., without requiring us to search for the declaration of that entity. The pattern-matching engine in our brains relies a great deal on these naming rules.

Consistency is more important than individual preferences so regardless of whether you find them sensible or not, the rules are the rules.

### General Naming Rules

Function names, variable names, and filenames should be descriptive; eschew abbreviation. Types and variables should be nouns, while functions should be "command" verbs.

#### How to Name

Give as descriptive a name as possible, within reason. Do not worry about saving horizontal space as it is far more important to make your code immediately understandable by a new reader. Examples of well-chosen names:

```cpp
int numberOfErrors;              // Good.
int countCompletedConnections;   // Good.
```

Poorly-chosen names use ambiguous abbreviations or arbitrary characters that do not convey meaning:

```cpp
int n;                           // Bad - meaningless.
int nerr;                        // Bad - ambiguous abbreviation.
int n_comp_conns;                // Bad - ambiguous abbreviation.
```

Type and variable names should typically be nouns: e.g., `FileOpener`, `NumErrors`.

Function names should typically be imperative (that is they should be commands): e.g., `OpenFile()`, `set_num_errors()`. There is an exception for accessors, which should be named the same as the variable they access.

#### Abbreviations

Do not use abbreviations unless they are extremely well known outside your project. For example:

```cpp
// Good
// These show proper names with no abbreviations.
int numberOfDnsConnections;  // Most people know what "DNS" stands for.
int priceCountReader;   // OK, price count. Makes sense.
```

```cpp
// Bad!
// Abbreviations can be confusing or ambiguous outside a small group.
int wgcconnections;  // Only your group knows what this stands for.
int pcreader;        // Lots of things can be abbreviated "pc".
```

Never abbreviate by leaving out letters:

```cpp
int error_count;  // Good.
```

```cpp
int error_cnt;    // Bad.
```

### File Names

Filenames should be all lowercase and can include underscores dashes (`-`). Do not use underscores in filenames (`_`).

#### Examples

Examples of acceptable file names:

```
my-useful-class.cpp
myusefulclass.cpp
myusefulclass-test.cpp
```

C++ files should end in `.cpp` and header files should end in `.h`.

Do not use filenames that already exist in `/usr/include`, such as `db.h`.

In general, make your filenames very specific. For example, use `http-server-logs.h` rather than `logs.h`. A class called `FooBar` should be declared in file called `foobar.h` and defined in `foobar.cpp`.

Inline functions must be in a `.h` file. If your inline functions are very short, they should go directly into your `.h` file. However, if your inline functions include a lot of code, they may go into a third file that ends in `-inl.h`. In a class with a lot of inline code, your class could have three files:

```cpp
foobar.h      // The class declaration.
foobar.cpp    // The class definition.
foobar-inl.h  // Inline functions that include lots of code.
```

### Type Names

Type names start with a capital letter and have a capital letter for each new word, with no underscores: `MyExcitingClass`, `MyExcitingEnum`. All type names are declared inside a namespace so no prefixing is used.

#### Examples

The names of all types; classes, structs, typedefs have the same naming convention - CamelCase with a leading capital letter, with no underscores. Enumerated type constants use the same convention as class constants, i.e. all uppercase with underscores. For example:

```cpp
// classes and structs
class UrlTable { ... };
class UrlTableTester { ... };
struct UrlTableProperties { ... };

// typedefs
typedef hash_map<UrlTableProperties *, string> PropertiesMap;

// enums
enum UrlTableErrors
{
  OK = 0,
  OUT_OF_MEMORY = 1,
  MALFORMED_INPUT = 2,
}
```

### Variable Names

Variable names should use camel case, with an initial lower case letter. Local variable names start with a lowercase letter. For example: `myExcitingLocalVariable`, Class member variables have prefix m. For example: `mMember`. The only exception to this rule is for public member variables in public structs where the prefix m is not required. Such member variables should also start with a lowercase letter. Avoid underscore characters in mixed case names. Constants should be all upper case, with underscores between words. Global variables should be avoided, however, if one is needed, prefix it with `g`.

#### Examples

For example:

```cpp
void SomeFunction()
{
  string tableName;  // OK - starts lowercase
  string tablename;  // OK - all lowercase.
}
```

```cpp
string TableName;   // Bad - starts with capital.
```

```cpp
class Foo
{
public:
  string mTable;     // OK
  string mTableName; // OK.
};
```

```cpp
struct Foo
{
  string table;     // OK
  string tableName; // OK.
};
```

```cpp
const int DAYS_IN_A_WEEK = 7;
```

```cpp
const int DaysInAWeek = 7; // Bad - constants should not use camel case.
const int DAYSINAWEEK = 7; // Bad - without underscore, words run together.
```

### Function Names

Regular function names are written in camel case starting with a capital letter and no underscores; accessors and mutators match the name of the variable with Get or Set as prefix.

#### Examples

```cpp
void MyExcitingFunction();
void MyExcitingMethod();

class MyClass
{
public:
  ...
  int GetNumEntries() const { return mNumEntries; }
  void SetNumEntries(int numEntries) { mNumEntries = numEntries; }

private:
  int mNumEntries;
};
```

```cpp
void my_boring_function(); // bad - uses underscores and no camel case.
```

#### Signal Accessors

Signal accessor methods are named `<Event>Signal()` and return a `<Event>SignalType&`, not `GetXSignal()` or `ConnectX()`:

```cpp
TouchEventSignalType&     TouchEventSignal();
SceneConnectedSignalType& SceneConnectedSignal();
ChildAddedSignalType&     ChildAddedSignal();
```

### Macro Names

Macros should not be used for programming, code should always be readable without preprocessor. Only allowed cases for macros are include guards, debug tracing and compile time feature flags **inside** .cpp files when no other variation mechanism is not possible. Always prefer variation through design and template mechanisms rather than `#define`. If you need to use macros, they're like this: `MY_MACRO_THAT_SCARES_SMALL_CHILDREN`.

#### Examples

```cpp
#define DEBUG_TRACE(__FILE__, __LINE__) ...
#ifndef ACTOR_H
#define ACTOR_H
...
#endif // ACTOR_H
```

#### Standard Assert and Log Macros

Do not use raw `assert()` or ad-hoc logging. Use the shared macro family instead:

- `DALI_ASSERT_ALWAYS(cond)` — always compiled in; use for conditions that must hold in release builds too.
- `DALI_ASSERT_DEBUG(cond)` — compiled out in release builds; use for internal invariants.
- `DALI_LOG_ERROR`, `DALI_LOG_ERROR_NOFN`, `DALI_LOG_RELEASE_INFO`, etc. — for diagnostic logging.

Combine the condition with `&& "message"` so the message appears in the assert/abort output:

```cpp
DALI_ASSERT_DEBUG((mOffScreenRenderableBitField & OFF_SCREEN_RENDERABLE_TYPE_FORWARD_MASK) != 0
                   && "Off screen renderable type forward not registered before!");
```

#### API Visibility Macros

Public classes carry a component-specific visibility macro directly after the `class` keyword, not a generic one — e.g. `DALI_CORE_API`, `DALI_ADAPTOR_API`, `DALI_TOOLKIT_API` depending on which library the class belongs to:

```cpp
class DALI_CORE_API Actor : public Handle
{
  ...
};
```

Each macro expands to `__attribute__((visibility("default")))` on GCC/Clang or `__declspec(dllexport/dllimport)` on Windows, defined once per component (e.g. `dali/public-api/common/dali-common.h` for dali-core).

## Comments

Comments are absolutely vital to keeping our code readable. The following rules describe what you should comment and where. But remember: while comments are very important, the best code is self-documenting. Giving sensible names to types and variables is much better than using obscure names that you must then explain through comments. When writing your comments, write for your audience: the next contributor who will need to understand your code.

### Comment Rules

#### Comment Style

API documentation must use doxygen comments: `/** */` or `///`

Internal comments can use `//` or `/* */` syntax, as long as you are consistent.

#### File Comments

Start each header file with a guard macro.

This should be the full namespace/path baked into an uppercase, underscore-separated
identifier: `DALI_<PATH>_<FILE>_H`. Do not use leading or trailing double underscores
(`__...__`) — those forms are reserved by the C++ standard for the implementation.

For example, a class in the Dali namespace called Actor, declared in `dali/public-api/actors/actor.h`, has the guard:

```cpp
#ifndef DALI_ACTOR_H
#define DALI_ACTOR_H
```

An Actor class in the Dali::Internal namespace, declared in `dali/internal/event/actors/actor-impl.h`, has the guard:

```cpp
#ifndef DALI_INTERNAL_ACTOR_H
#define DALI_INTERNAL_ACTOR_H
```

The copyright & legal notice should follow the guard macro (inside the #ifdef), since this will improve build times. Start each source file with the copyright & legal notice. Use the standard Apache-2.0 boilerplate, updating only the year:

```cpp
#ifndef DALI_ACTOR_H
#define DALI_ACTOR_H

/*
 * Copyright (c) 2026 Samsung Electronics Co., Ltd.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 *
 */
```

#### Include Grouping

After the license block, group includes under fixed comment banners, in this order: the file's own class header (in a `.cpp` file), then external (system/third-party) includes, then internal (project) includes:

```cpp
// CLASS HEADER
#include <dali/public-api/actors/actor.h>

// EXTERNAL INCLUDES
#include <cstdint>

// INTERNAL INCLUDES
#include <dali/internal/event/common/object-impl.h>
```

#### Class Comments

Every class definition should have an accompanying comment that describes what it is for and how it should be used. It should have a brief description, followed by a newline and a broader description.

```cpp
/**
 * @brief Iterates over the contents of a GargantuanTable.
 *
 * Example usage:
 *    GargantuanTableIterator* iter = table->NewIterator();
 *    for(iter->Seek("foo"); !iter->done(); iter->Next())
 *    {
 *      process(iter->key(), iter->value());
 *    }
 *    delete iter;
 */
class GargantuanTableIterator
{
  ...
};
```

Document the synchronization assumptions the class makes, if any. If an instance of the class can be accessed by multiple threads, take extra care to document the rules and invariants surrounding multithreaded use.

#### Function Comments

Every function declaration in a header file must have a brief comment that describes what the function does, followed by a newline. It may be followed by a detailed description on how to use it. These comments should be descriptive ("Opens the file") rather than imperative ("Open the file"); the comment describes the function, it does not tell the function what to do. Each parameter must be documented, as must any return value.

Types of things to mention in comments at the function declaration:

- What the inputs and outputs are.
- For class member functions: whether the object remembers reference arguments beyond the duration of the method call, and whether it will free them or not.
- If the function allocates memory that the caller must free.
- Whether any of the arguments can be `NULL`
- If there are any performance implications of how a function is used.
- If the function is re-entrant. What are its synchronization assumptions?

```cpp
/**
 * @brief Get the iterator for this data table.
 *
 * It is the client's responsibility to delete the iterator,
 * and it must not use the iterator once the GargantuanTable object
 * on which the iterator was created has been deleted.
 *
 * The iterator is initially positioned at the beginning of the table.
 *
 * This method is equivalent to:
 *    Iterator* iter = table->NewIterator();
 *    iter->Seek("");
 *    return iter;
 * If you are going to immediately seek to another place in the
 * returned iterator, it will be faster to use NewIterator()
 * and avoid the extra seek.
 * @return an iterator for this table.
 */
Iterator* GetIterator() const;
```

When commenting constructors and destructors, remember that the person reading your code knows what constructors and destructors are for, so comments that just say something like "destroys this object" are not useful.

#### Variable Comments

In general the actual name of the variable should be descriptive enough to give a good idea of what the variable is used for. In certain cases, more comments are required - use Doxygen formatting.

```cpp
private:
  /**
   * @brief Keeps track of the total number of entries in the table.
   *
   * Used to ensure we do not go over the limit. -1 means
   * that we don't yet know how many entries the table has.
   */
  int mNumTotalEntries;

  float mDuration; ///< Duration in seconds.
```

#### Duplicate documentation

Try to avoid duplicate documentation by using the @copydoc command.

`@copydoc` copies a documentation block from the object specified by link-object and pastes it at the location of the command. This command can be useful to avoid cases where a documentation block would otherwise have to be duplicated or it can be used to extend the documentation of an inherited member.

```cpp
/**
 * @copydoc MyClass::myfunction()
 * More documentation.
 */
```

If the member is overloaded, you should specify the argument types explicitly (without spaces!), like in the following:

```cpp
/** @copydoc MyClass::myfunction(type1,type2) */
```

#### Punctuation, Spelling and Grammar

Pay attention to punctuation, spelling, and grammar; it is easier to read well-written comments than badly written ones.

Comments should usually be written as complete sentences with proper capitalization and periods at the end. Shorter comments, such as comments at the end of a line of code, can sometimes be less formal, but you should be consistent with your style. Complete sentences are more readable, and they provide some assurance that the comment is complete and not an unfinished thought.

#### TODO Comments

Use `TODO` comments for code that is temporary, a short-term solution, or good-enough but not perfect.

#### Deprecation Comments

Mark deprecated interface points with `@deprecated` comments.

#### Doxygen Tags

The following order should be followed when using doxygen tags for functions, classes, structs etc.

```cpp
/**
 * @deprecated DALi X.X.X    Mandatory, if applicable    // Version deprecated and alternative
 * @brief                    Mandatory                   // Explain the API briefly
 * @details                  Optional                    // Explain the API in more detail. Use this tag or add more
 *                                                       // information after a blank line following the @brief
 * @SINCE_X.X.X              Mandatory                   // Version added
 * @param[in]                Mandatory, if applicable    // In Parameter list
 * @param[out]               Mandatory, if applicable    // Out Parameter list
 * @param[in,out]            Mandatory, if applicable    // In/Out Parameter list
 * @return                   Mandatory, if applicable    // Return value
 * @pre                      Optional                    // Pre-condition
 * @post                     Optional                    // Post-condition
 * @note                     Optional                    // Any notes that apply
 * @see                      Optional                    // Other related APIs
 */
```

Where X.X.X is the next release version (ensure you're patch gets merged before the release), e.g. `@SINCE_1_0.0`.

#### Doxygen Grouping

Every `public-api/` or `devel-api/` header should open its doxygen block with an `@addtogroup` tag naming its module, and close the group with `@}` at the end of the file:

```cpp
/**
 * @addtogroup dali_core_actors
 * @{
 */
...
/**
 * @}
 */
```

Folder location also communicates API stability: `public-api/` is the stable, released API; `devel-api/` is still under development and may change without notice; `internal/` is implementation detail and is never exposed to doxygen or clients.

## Formatting

A project is much easier to follow if everyone uses the same style. It is important that all contributors follow the style rules so that they can all read and understand everyone's code easily.

### Formatting Rules

#### Basic Rules

- Use only spaces, and indent 2 spaces at a time
- Avoid unnecessary trailing whitescape
- Avoid overly long lines, modern screens and editors can handle upto 120 characters
- Use UTF-8 formatting for non-ASCII characters
- Do not put a space just inside parentheses, and no space between a keyword (`if`, `for`, `while`, `switch`) or function name and its opening parenthesis.
- Braces must each be in a line on their own.

Hex encoding is also OK, and encouraged where it enhances readability — for example, `"\xEF\xBB\xBF"` is the Unicode zero-width no-break space character, which would be invisible if included in the source as straight UTF-8.

#### Class Format

Sections in `public`, `protected` and `private` order. Typedefs and Enums first, Constructor(s) & Destructors & public API next, then virtual methods and implementation details last.

```cpp
/**
 * @brief Class purpose.
 *
 * Long description.
 */
class MyClass : public Base
{
public: // API

  /**
   * @brief Short description.
   *
   * Documentation
   */
  MyClass();  // 2 space indent.

  /**
   * @brief Short description.
   *
   * @param[in] var Description
   */
  explicit MyClass(int var);

  /**
   * @brief Short description.
   *
   * Documentation
   */
  virtual ~MyClass();

  /**
   * @brief Short description.
   *
   * Documentation
   */
  void SomeFunction();

  /**
   * @brief Short description.
   *
   * Documentation
   */
  void SomeFunctionThatDoesNothing();

  /**
   * @brief Short description.
   *
   * Documentation
   * @param[in] var Parameter description
   */
  void SetSomeVar(int var);

  /**
   * @brief Short description.
   *
   * Documentation.
   * @return value description.
   */
  int GetSomeVar() const;

private: // Implementation

  MyClass(MyClass& aRHs); ///< no copying
  MyClass& operator=(const MyClass& aRHs);  ///< no copying.
  bool SomeInternalFunction();

  int mSomeVar;       ///< short description.
  int mSomeOtherVar;  ///< short description.
};
```

#### Constructor Initializer Lists

Constructor initializer lists can be all on one line or with subsequent lines indented. Always initialize things in declaration order, base class first!!

```cpp
// When it all fits on one line:
MyClass::MyClass(int var) : Base(var), mSomeVar(var), mSomeOtherVar(var + 1)
{
}

// When it requires multiple lines, indent, putting the colon on
// the first initializer line:
MyClass::MyClass(int var)
: Base(var),
  mSomeVar(var),
  mSomeOtherVar(var + 1)
{
  DoSomething();
}
```

#### Namespace Formatting

The contents of namespaces are not indented.

```cpp
namespace
{

void Foo()
{  // Correct.  No extra indentation within namespace.
  ...
}

}  // namespace
```

When declaring nested namespaces, put each namespace on its own line.

```cpp
/**
 * @brief description of namespace
 */
namespace Foo
{

/**
 * @brief description of namespace
 */
namespace Bar
{
```

#### Function Declarations and Definitions

Return type on the same line as function name, parameters on the same line if they fit. All parameters should be named, with identical names in the declaration and definition.

```cpp
ReturnType ClassName::FunctionName(Type parameterName1, Type parameterName2)
{
  DoSomething();
}
```

If you have too much text to fit on one line:

```cpp
ReturnType ClassName::ReallyLongFunctionName(Type parameterName1,
                                              Type parameterName2,
                                              Type parameterName3)
{
  DoSomething();
}
```

or if you cannot fit even the first parameter:

```cpp
ReturnType LongClassName::ReallyReallyReallyLongFunctionName(
  Type parameterName1,  // 2 space indent
  Type parameterName2,
  Type parameterName3)
{
  DoSomething();  // 2 space indent
  ...
}
```

`const` keyword should be on the same line as the last parameter:

```cpp
// Everything in this function signature fits on a single line
ReturnType FunctionName(Type parameter) const
{
  ...
}

// This function signature requires multiple lines, but
// the const keyword is on the line with the last parameter.
ReturnType ReallyLongFunctionName(Type parameter1,
                                   Type parameter2) const
{
  ...
}
```

If some parameters are unused, comment out the variable name in the function definition:

```cpp
// Always have named parameters in interfaces.
class Shape
{
public:
  virtual void Rotate(double radians) = 0;
}

// Always have named parameters in the declaration.
class Circle : public Shape
{
public:
  virtual void Rotate(double radians);
}

// Comment out unused named parameters in definitions.
void Circle::Rotate(double /*radians*/)
{
}
```

```cpp
// Bad - if someone wants to implement later, it's not clear what the
// variable means.
void Circle::Rotate(double)
{
}
```

#### Function Calls

On one line if it fits; otherwise, wrap arguments at the parenthesis. Do not put a space just inside the parentheses.

```cpp
bool retval = DoSomething(argument1, argument2, argument3);
```

If the arguments do not all fit on one line, they should be broken up onto multiple lines, with each subsequent line aligned with the first argument:

```cpp
bool retval = DoSomething(aVeryVeryVeryVeryLongArgument1,
                           argument2, argument3);
```

If the function has many arguments, consider having one per line if this makes the code more readable:

```cpp
bool retval = DoSomething(argument1,
                           argument2,
                           argument3,
                           argument4);
```

If the function signature is so long that it cannot fit, place all arguments on subsequent lines:

```cpp
if(...)
{
  DoSomethingThatRequiresALongFunctionName(
    aVeryLongArgument1,  // indent
    argument2,
    argument3,
    argument4);
}
```

#### Conditionals

The `if` statement should be followed only by the conditional, with no space between `if` and the opening parenthesis, and no space just inside the parentheses. The following clause must always be surrounded by braces on separate lines. The `else` keyword belongs on its own line, and the following clause must always be surrounded by braces.

```cpp
if(condition) // no space before ( or just inside the parentheses
{
  ...
}
else
{
  ...
}
```

```cpp
if( condition )   // Bad - spaces around the condition are no longer used.
if (condition)    // Bad - space between "if" and "(".
if(condition){    // Bad. Mis-aligned parentheses are harder to read.
```

Must always have curly braces for the clauses, and they must always be on a line on their own:

```cpp
if(condition)
{
  DoSomething();  // 2 space indent.
}
else
{
  DoSomethingElse(); // 2 space indent.
}
```

#### Loops and Switch Statements

Switch statements use braces for blocks. Empty loop bodies must use `{}` or `continue`. Switch statements should always have a `default` case unless they switch on an `enum` type, for which the compiler will ensure all values are handled.

If the default case should never execute, simply `assert`. Do not put a space before the opening parenthesis or just inside it.

`case` blocks in `switch` statements must have curly braces. They should be placed as shown below.

```cpp
switch(var)
{
  case 0: // 2 space indent
  {
    ...      // 2 space indent
    break;
  }
  case 1:
  {
    ...
    break;
  }
  default:
  {
    DALI_ASSERT_ALWAYS(false);
  }
}
```

Empty loop bodies should use `{}` or `continue`, but not a single semicolon.

```cpp
while(condition)
{
  // Repeat test until it returns false.
}

for(int i = 0; i < someNumber; ++i)
{
}  // Good - empty body, clearly separated.

while(condition)
  continue;  // Good - continue indicates no logic.
```

```cpp
while(condition);  // Bad - looks like part of do/while loop.
```

#### Pointer and Reference Expressions

No spaces around period or arrow. Pointer operators do not have trailing spaces.

The following are examples of correctly-formatted pointer and reference expressions:

```cpp
x = *p;
p = &x;
x = r.y;
x = r->y;
```

When declaring a pointer variable or argument, you may place the asterisk adjacent to either the type or to the variable name, however, placing with the type is preferred.

```cpp
// These are fine, space preceding.
char *c;
const string &str;

// These are fine, space following.
char* c;    // but remember to do each variable on its own line or "char* c, *d, *e, ...;"!
const string& str;
```

```cpp
char * c;  // Bad - spaces on both sides of *
const string & str;  // Bad - spaces on both sides of &
```

You should do this consistently within a single file, so, when modifying an existing file, use the style in that file.

#### Boolean Expressions

When you have a boolean expression that is long, be consistent in how you break up the lines.

In this example, the logical AND operator is always at the end of the lines:

```cpp
if(thisOneThing > thisOtherThing &&
   aThirdThing == aFourthThing &&
   yetAnother && lastOne)
{
  ...
}
```

#### Return Values

Do not needlessly surround the `return` expression with parentheses.

```cpp
return result;                 // No parentheses in the simple case.
return (someLongCondition &&   // Parentheses ok to make a complex
        anotherCondition);     //     expression more readable.
```

```cpp
return (value);                // You wouldn't write var = (value);
return(result);                // return is not a function!
```

#### Variable and Array Initialization

Your choice of `=` or `()`.

You may choose between `=` and `()`; the following are all correct:

```cpp
int x = 3;
int x(3);
string name("Some Name");
string name = "Some Name";
```

#### Preprocessor Directives

Preprocessor directives should not be indented but should instead start at the beginning of the line.

Even when pre-processor directives are within the body of indented code, the directives should start at the beginning of the line.

```cpp
// Good - directives at beginning of line
  if(lopsidedScore)
  {
#if DISASTER_PENDING      // Correct -- Starts at beginning of line
    DropEverything();
#endif
    BackToNormal();
  }
```

```cpp
// Bad - indented directives
  if(lopsidedScore)
  {
    #if DISASTER_PENDING  // Wrong!  The "#if" should be at beginning of line
    DropEverything();
    #endif                // Wrong!  Do not indent "#endif"
    BackToNormal();
  }
```

---

That's all folks, if you read this far you are now all equipped to write good code! :)
