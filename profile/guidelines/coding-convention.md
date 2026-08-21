# Dali C++ Coding Conventions

## General coding conventions

### Type casting

**Never use C-Style Casts**

The C-style cast - "(type) expr" used to convert one fundamental type to another is subject to implementation-defined effects. For scalar types it can result in silent truncation of the value. For pointers and references, it does not check the compatibility of the value with the target type.

**Don't cast away const, use mutable keyword instead**

**Don't reinterpret_cast, fix the design instead**

**Remember that reference cast will throw an error if cast fails**

**Avoid using pointer or reference casts. They have been referred to as the goto of OO programming, fix the design instead**

```cpp
X* ptr = static_cast<X*>(y_ptr); // ok, compiler checks whether types are compatible
```

```cpp
(Foo*) ptr; // bad! C-cast is not guaranteed to check and never complains
```

### Public API vs Devel API

Folder structure encodes API stability, and must be paired with matching doxygen tags:

- `public-api/` — stable, released API. Every member is tagged `@SINCE_x.y.z` with the DALi
  version it was introduced in.
- `devel-api/` — API still under development; may change without notice.
- `internal/` — implementation detail; never exposed to doxygen or to clients.

Public and devel headers open their doxygen block with an `@addtogroup <module_name>` tag and
close it with `@}` — see the [Doxygen Grouping](coding-style.md#doxygen-grouping) section of
the coding style doc.

## Classes

A class interface should be complete and minimal. Class should encapsulate one thing and one thing only. A complete interface allows clients to do anything they may reasonably want to do. On the other hand, a minimal interface will contain as few functions as possible. Class methods must be defined in the same order as they are declared. This helps navigating through code.

### Compulsory member functions

Every class must define default constructor, copy constructor, assignment operator and destructor. If you dont declare them, compiler will and the compiler generated versions are usually not good or safe enough. If your class does not support copying, then declare copy constructor and assignment operator as private and don't define them.

If for example the assignment operator is not needed for a particular class, then it must be declared private and not defined. Any attempt to invoke the operator will result in a compile-time error. On the contrary, if the assignment operator is not declared, then when it is invoked, a compiler-generated form will be created and subsequently executed. This could lead to unexpected results. The same goes with default constructor and copy constructor.

```cpp
class X
{
  X();                     // default constructor
  X(const X&);             // copy constructor
  X& operator=(const X&);  // copy assignment operator
  ~X();                    // destructor
};

class X
{
  X();                               // default constructor
  ~X();                              // destructor
  X(const X&) = delete;              // copy constructor not allowed
  X& operator=(const X&) = delete;   // copy assignment operator not allowed
};
```

### Class types

Classes can have either **Value** semantics or **Pointer** semantics. Not both. It must be clearly documented whether a class follows value or pointer semantics and this also sets requirements on the class interface.

Classes with **Value** semantics are passed as value types. These classes provide a copy constructor, a default constructor and assignment operator so that they can be copied and also stored on STL containers.

Classes with **Pointer** semantics are always passed through pointers, references or smart pointers. These classes are ususally compound types that cannot be easily copied and thus prevent copy constructor, and assignment operator. They can be only stored on STL containers through smart pointers.

### Handle/Body Idiom

Public API classes (e.g. `Dali::Actor`) follow Value semantics and are thin handles wrapping
a reference-counted internal object (e.g. `Dali::Internal::Actor`), rather than exposing the
implementation directly. This is how DALi implements the value-semantics rule above while
still sharing state cheaply between handle copies.

Internal access from within the library is via a pair of free-function overloads named
`GetImplementation()`, declared alongside the internal class, not as a member function:

```cpp
inline Internal::Actor&       GetImplementation(Dali::Actor& actor);
inline const Internal::Actor& GetImplementation(const Dali::Actor& actor);
```

Each overload asserts the handle isn't empty (`DALI_ASSERT_ALWAYS(actor && "Actor handle is empty")`)
before returning a reference to the underlying implementation object. Every public handle type
in DALi follows this same pattern.

### Access Rights

Public and protected data should only be used in structs, not classes. Roughly two types of classes exist: those that essentially aggregate data and those that provide an abstraction while maintaining a well-defined state or invariant.

A structure should be used to model an entity that does not require an invariant (Plain Old Data)
A class should be used to model an entity that maintains an invariant.

**Rationale:** A class is able to maintain its invariant by controlling access to its data. However, a class cannot control access to its members if those members non-private. Hence all data in a class should be private.

### Constructors

Virtual function calls are not allowed from constructor. Rationale: Virtual functions are resolved statically (not dynamically) in constructor.

Member initialization order must be the same in which they are declared in the class. Note: Since base class members are initialized before derived class members, base class initializers should appear at the beginning of the member initialization list.

**Rationale:** Members of a class are initialized in the order in which they are declared—not the order in which they appear in the initialization list.

Constructor body should not throw an exception, keep constructor simple and trivial. If constructor fails, objects lifecycle never started, destructor will not be called.

Declare all single argument constructors as explicit thus preventing their use as implicit type convertors.

```cpp
class C
{
public:
  explicit C(int);          // good, explicit
  C(int, int);              // ok more than one non-default argument
};
```

```cpp
class C
{
public:
  C(double);                // bad, can be used in implicit conversion
  C(float f, int i = 0);    // bad, implicit conversion constructor
  C(int i = 0, float f = 0.0); // bad, default constructor, but also a conversion constructor
};
```

### Destructor

All classes should define a destructor, either:

- public for value types
- public and virtual for base classes with virtual methods
- protected and virtual for base classes to prevent deletion (and ownership) through base class

This prevents undefined behavior. If an application attempts to delete a derived class object through a base class pointer, the result is undefined if the base class destructor is non-virtual.

Virtual function calls are not allowed from inside the destructor. Rationale: A class's virtual functions are resolved statically (not dynamically) in its destructor.

All resources acquired by a class shall be released by the class's destructor.

Destructor is not allowed to throw an exception, avoid doing complicated things in destructor.

### Methods

Don't shortcut, like use the returned reference of getter to assign a new value. If a Setter is missing, add it!

```cpp
initial.GetPosition() = Position(10, 10); // bad!, If GetPosition is one day changed to return copy
                                          // of Position this code silently changes to a no-op.
```

```cpp
initial.SetPosition(Position(10, 10));
```

Code that is not used (commented out) should be deleted. Rationale: No dead code should be left to confuse other people. Exception: Code that is simply part of an explanation may appear in comments.

### Inline member functions

GCC automatically inlines member functions defined within the class body of C++ programs even if they are not explicitly declared inline.

```cpp
class Color
{
  inline float& GetRed()  { return mRed;   } // inline keyword not needed
  inline float& GetGreen(){ return mGreen; }
};
```

```cpp
class Color
{
  float& GetRed()   { return mRed;   }
  float& GetGreen(){ return mGreen; }
};
```

If there are a lot of inlines, they should be in a .inl file. Remember the inline keyword is just a hint to the compiler. Whether a function will be inlined or not is down to the compiler and its flags.

### Conversion operators

Don't declare implicit conversion operators in classes. They allow the compiler to trip you up and go from one type to another via the conversion operator unintentionally. Conversion operators are particularly dangerous in conjunction with auto keyword. If conversion is required, make it explicit or better yet, add a getter with a more meaningfull name.

```cpp
// Bad:
class SmallInt
{
public:
  // implicit conversion to float
  operator float() const { return float(val); }
private:
  int val;
};
//... and in the program:

int main(void)
{
  int value;
  SmallValue foo;
  value = foo; // oops, didn't really want to allow conversion to int but the compiler can do that as float can be assigned to int.

  return 0;
}
```

```cpp
// Good:
class SmallInt
{
public:
  // explicit getter for float
  float AsFloat const { return static_cast<float>(val); }
private:
  int val;
};
//... and in the program:

int main(void)
{
  int value;
  SmallValue foo;
  si.AsFloat() + 3; // ok: explicitly request the conversion

  return 0;
}

// Good:
class SmallInt
{
public:
  // explicit conversion to int
  explicit operator int() const { return val; }
private:
  int val;
};
//... and in the program:

int main()
{
  SmallInt si = 3; // ok: the SmallInt constructor is not explicit
  si + 3; // error: implicit is conversion required, but operator int is explicit
  static_cast<int>(si) + 3; // ok: explicitly request the conversion

  return 0;
}
```

### Auto keyword

auto keyword should only be used where it improves the readability of the code and does not lead to ambiguities. Never use auto in a line where multiple different types occur as part of expressions like additions, subtracts, multiplies as the conversion ordering rules are not always obvious.

```cpp
// Good:
auto actor = Actor::DownCast(GetOwner()); // it is obvious that actor is of type Actor so no need to retype the type
auto widthMode = widthMeasureSpec.GetMode(); // it is relatively obvious that Mode is an enumeration with potentially long name so no need to repeat the type, no ambiguity
auto childLayout = GetChildAt(i); // name of the variable is clear enough indication of the type, no ambiguity
auto childPosition = childOwner.GetProperty<Dali::Vector3>(Actor::Property::POSITION); // getter already contains the type, no need to repeat it

for(auto&& renderTask : mImpl->taskList.GetTasks()) // iterator type not relevant for the algorithm, code much cleaner with auto
{
  renderTask->UpdateState();
}
```

```cpp
// Bad:
auto width = layout->GetWidth() - padding.end - padding.start; // not obvious what the final type ends up as multiple type conversions may occur

auto size = std::max(LayoutLength(0), specSize - padding); // not obvious which of the types is preferred by compiler; or what the type of specSize - padding actually is

auto minPosition = Vector3(Vector3::ZERO); // auto does not add any value here
```

```cpp
// Good:
Vector3 minPosition; // vector initializes to 0,0,0
```

```cpp
// Bad:
auto specification = MeasureSpec(GetMeasuredHeight(), MeasureSpec::Mode::EXACTLY); // no value in typing auto in assignment, much cleaner and less ambiguous to write:
```

```cpp
// Good:
MeasureSpec specification(GetMeasuredHeight(), MeasureSpec::Mode::EXACTLY); // obvious construction of a type with parameters
```

### Class Inheritance

#### Overriding

When using inheritance, any methods in the base class that can be overridden MUST be marked as **virtual**. In deriving classes, when a virtual method is overridden, then only use the **override** keyword. If a method should not be overridden further, then use the **final** keyword alone.

```cpp
// Good:
class Base
{
public:
  virtual void Print() const;
  virtual void SetPrintSpeed(float speed);
};

class Derived : public Base
{
public:
  void Print() const override;
  void SetPrintSpeed(float speed) final;
};
```

If a class should not be overridden then use the **final** keyword on the class itself. This should also be done for a derived class that should not to be overridden further as well. Overridden methods within that class can be marked as **final** or **override**.

```cpp
class Derived final : public Base
{
public:
  void Print() const override;
  void SetPrintSpeed(float speed) final;
};
```

#### Overloading

Overloading of Base class methods SHOULD be avoided but if it's required, then use the **using** keyword.

```cpp
class Derived : public Base
{
public:
  void Print(float number) const; // overloaded member
  using Base::Print; // Make the Base class' Print method visible here as well.
};
```

If we do not add the using line, then we can only use the overloaded Print method for a Derived object (unless we cast to the Base class). Attempting to use the base class' Print() method on a Derived object would result in a compilation error.

## Toolkit-Specific Conventions

These conventions apply specifically to UI controls and visuals in dali-toolkit.

### Control / Visual Construction Pattern

- **Controls**: `public-api/controls/<name>.h` declares the handle class with a static
  factory `<Name>::New(...)`. `internal/controls/<group>/<name>-impl.{h,cpp}` holds the
  corresponding `Internal::<Name>` implementation (see [Handle/Body Idiom](#handlebody-idiom)).
- **Visuals**: concrete visual classes are named `<Feature>Visual` (e.g. `ImageVisual`,
  `ColorVisual`), each paired with an `IntrusivePtr` typedef named `<Feature>VisualPtr`, and a
  static `New(...)` factory returning that typedef, e.g.:

  ```cpp
  typedef IntrusivePtr<ImageVisual> ImageVisualPtr;
  static ImageVisualPtr New(VisualFactoryCache& factoryCache, ...);
  ```

  The shared base is `Visual::Base` / `Visual::BasePtr` — not `VisualBase`.

### Property Registration

Every `Toolkit::Control`-derived impl `.cpp` registers its type and properties with a fixed
macro block, vertically aligned:

```cpp
DALI_TYPE_REGISTRATION_BEGIN(Toolkit::TextLabel, Toolkit::Control, Create);
DALI_PROPERTY_REGISTRATION(Toolkit, TextLabel, "text",       STRING, TEXT)
DALI_PROPERTY_REGISTRATION(Toolkit, TextLabel, "fontFamily", STRING, FONT_FAMILY)
...
```

## General design principles

Here's a few pragmatic programmer guidelines to follow ([Web version](http://www.codinghorror.com/blog/files/Pragmatic%20Quick%20Reference.htm))

### Design Principles

- **Care About the Software, Care about your API users and end users** — Why spend your life developing software unless you care about doing it well? Turn off the autopilot and take control. Constantly critique and appraise your work.

- **Don't Live with Broken Windows** — Fix bad designs, wrong decisions, and poor code when you see them. You can't force change on people. Instead, show them how the future might be and help them participate in creating it.

- **Remember the Big Picture** — Don't get so engrossed in the details that you forget to check what's happening around you.

- **DRY - Don't Repeat Yourself** — Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

- **Eliminate Effects Between Unrelated Things** — Design components that are self-contained. independent, and have a single, well-defined purpose.

- **There Are No Final Decisions** — No decision is cast in stone. Instead, consider each as being written in the sand at the beach, and plan for change.

- **Fix the Problem, Not the Blame** — It doesn't really matter whether the bug is your fault or someone else's—it is still your problem, and it still needs to be fixed.

- **You Can't Write Perfect Software** — Software can't be perfect. Protect your code and users from the inevitable errors.

- **Design with Contracts** — Use contracts to document and verify that code does no more and no less than it claims to do.

- **Crash Early** — A dead program normally does a lot less damage than a crippled one.

- **Use Assertions to Prevent the Impossible** — Assertions validate your assumptions. Use them to protect your code from an uncertain world.

- **Use Exceptions for Exceptional Problems** — Exceptions can suffer from all the readability and maintainability problems of classic spaghetti code. Reserve exceptions for exceptional things.

- **Minimize Coupling Between Modules** — Avoid coupling by writing "shy" code and applying the Law of Demeter.

- **Put Abstractions in Code, Details in Metadata** — Program for the general case, and put the specifics outside the compiled code base.

- **Always Design for Concurrency** — Allow for concurrency, and you'll design cleaner interfaces with fewer assumptions.

- **Don't Program by Coincidence** — Rely only on reliable things. Beware of accidental complexity, and don't confuse a happy coincidence with a purposeful plan.

- **Test Your Estimates** — Mathematical analysis of algorithms doesn't tell you everything. Try timing your code in its target environment.

- **Refactor Early, Refactor Often** — Just as you might weed and rearrange a garden, rewrite, rework, and re-architect code when it needs it. Fix the root of the problem.

- **Design to Test** — Start thinking about testing before you write a line of code.

- **Abstractions Live Longer than Details** — Invest in the abstraction, not the implementation. Abstractions can survive the barrage of changes from different implementations and new technologies.

- **Coding Ain't Done 'Til All the Tests Run** — 'Nuff said.

- **Use Saboteurs to Test Your Testing** — Introduce bugs on purpose in a separate copy of the source to verify that testing will catch them.

- **Find Bugs Once** — Once a human tester finds a bug, it should be the last time a human tester finds that bug. Automatic tests should check for it from then on.

- **Sign Your Work** — Craftsmen of an earlier age were proud to sign their work. You should be, too.

### Avoid Tight Coupling

Always choose the loosest possible coupling between entities. In C++ the tightest coupling is Friend, second is Inheritance, then Containment and last is Usage through reference, pointer or handle.

- Friend defines a "must-know" about details of implementation, don't use it unless your happy stating that Xxx really **must** know about Yyy implementation. and Yyy can never change without informing Xxx.

- Inheritance defines a "is-a" relationship, don't use it unless you really can naturally say Xxx is-a Yyy. Most of the cases containment through interface is what you need.

- Containment defines a "owns-a" relationship, use it when you have a natural Xxx owns-a Yyy relationship.

Most of the time containment through interface and normal usage is what you should go for. Strong ownership always beats sharing through reference counting. Reference counting means "part owns". You would not want to part own anything in real life, so why do that in software? Sooner or later it will leak.

> **Note:** despite this guidance, reference counting is DALi's standard ownership idiom for
> internal/handle objects (see [Reference Counting and Smart Pointers](#reference-counting-and-smart-pointers)
> below) — prefer it there over ad-hoc ownership, but do not introduce reference counting for
> new designs where a single, strongly-owning relationship (containment) is possible instead.

### Reference Counting and Smart Pointers

DALi does not use `std::shared_ptr` / `std::unique_ptr` for its object model. Instead:

- `Dali::RefObject` — a base class providing `Reference()` / `Unreference()`; an object
  self-destructs once its count reaches zero.
- `Dali::IntrusivePtr<T>` — the corresponding smart pointer, used wherever a `RefObject`-derived
  type needs shared ownership.

Every internal/impl class (reached via the [Handle/Body idiom](#handlebody-idiom) above) derives
from `RefObject` and is held by `IntrusivePtr`, including in dali-adaptor and dali-toolkit — e.g.
`Visual::BasePtr` is `IntrusivePtr<Visual::Base>`. Treat `RefObject`/`IntrusivePtr` as the required
tool for this kind of ownership, not `std::shared_ptr`.

### Open Closed Principle

Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification. That is, such an entity can allow its behaviour to be modified without altering its source code.

### Dependency Inversion Principle

High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend upon details. Details should depend upon abstractions.

---

That's all folks, if you read this far you are now all equipped to write good code! :)
