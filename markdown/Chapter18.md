# Visiting with the New C++ Standard

[TOC]

## C++11 Features Revisited

### New Types

C++11 adds the `long long` and `unsigned long long` types to support 64-bit integers (or wider) and the `char16_t` and `char32_t` types to support 16-bit and 32-bit character representations, respectively. It also adds the “raw” string. Chapter 3,“Dealing with Data,” discusses these additions.

### Uniform Initialization

C++11 extends the applicability of the brace-enclosed list (**list-initialization**) so that it can be used with all built-in types and with user-defined types (that is, class objects).The list can be used either with or without the `=` sign:

```c++
int x = {5};
double y {2.75};
short quar[5] {4,5,2,76,1};
```

Also the list-initialization syntax can be used in new expressions:

```c++
int * ar = new int [4] {2,4,6,7}; // C++11
```

With class objects, a braced list can be used instead of a parenthesized list to invoke a constructor:

```c++
class Stump 
{
private:
	int roots;
	double weight; 
public:
Stump(int r, double w) : roots(r), weight(w) {} 
};
Stump s1(3,15.6); 		// old style 
Stump s2{5, 43.4}; 		// C++11 
Stump s3 = {4, 32.1}; 	// C++11
```

#### Narrowing

The initialization-list syntax provides protection against narrowing—that is, against assigning a numeric value to a numeric type not capable of holding that value. Ordinary initialization allows you to do things that may or may not make sense:

```c++
char c1 = 1.57e27; // double-to-char, undefined behavior 
char c2 = 459585821; // int-to-char, undefined behavior
```

If you use initialization-list syntax, however, the compiler disallows type conversions that attempt to store values in a type “narrower” than the value:

```c++
char c1 {1.57e27}; // double-to-char, compile-time error
char c2 = {459585821};// int-to-char,out of range, compile-time error
```

#### `std::initializer_list`

The STL containers provide constructors with `initializer_list` arguments:

```c++
vector<int> a1(10); // uninitialized vector with 10 elements 
vector<int> a2{10}; // initializer-list, a2 has 1 element set to 10 
vector<int> a3{4,6,1}; // 3 elements set to 4,6,1
```

### Declarations

#### `auto`

C++11 strips the keyword `auto` of its former meaning as a storage class specifier (Chapter 9,“Memory Models and Namespaces”) and puts it to use (Chapter 3) to imple- ment automatic type deduction, provided that an explicit initializer is given.

```c++
auto maton = 112; // maton is type int
auto pt = &maton; // pt is type int *
double fm(double, int);
auto pf = fm; // pf is type double (*)(double,int)
```

The `auto` keyword can simplify template declarations too. For example, if `il` is an object of type `std::initializer_list<double>`, you can replace

```c++
for (std::initializer_list<double>::iterator p = il.begin();
											p !=il.end(); p++)
```

with this

```c++
for (auto p = il.begin(); p !=il.end(); p++)
```

#### `decltype`

The decltype keyword creates a variable of the type indicated by an expression.The fol- lowing statement means “make `y` the same type as `x`,” where x is an expression:

```c++
decltype(x) y;
```

#### Trailing Return Type

C++11 introduces a new syntax for declaring functions, one in which the return type comes after the function name and parameter list instead of before them:

```c++
double f1(double, int); 		// traditional syntax
auto f2(double, int) -> double; // new syntax, return type is double
```

The new syntax may look like a step backwards in readability for the usual function declarations, but it does make it possible to use decltype to specify template function return types:

```c++
template<typename T, typename U) 
auto eff(T t, U u) -> decltype(T*U) 
{
	... 
}
```

Template Aliases: `using =`

It’s handy to be able to create aliases for long or complex type identifiers. C++ already had `typedef` for that purpose:

```c++
typedef std::vector<std::string>::iterator itType;
```

C++11 provides a second syntax (discussed in Chapter 14, ” Reusing Code in C++”) for creating aliases:

```c++
using itType = std::vector<std::string>::iterator;
```

The difference is that the new syntax also can be used for partial template specializations, whereas `typedef` can’t:

```c++
template<typename T>
	using arr12 = std::array<T,12>; // template for multiple aliases
```

#### `nullptr`

The null pointer is a pointer guaranteed not to point to valid data.Traditionally, C++ has represented this pointer in source code with `0`, although the internal representation could be different.This raises some problems because it makes 0 both a pointer constant and an integer constant.For example, the value 0 could be passed to a function accepting an int argument, but the compiler will identify an attempt to pass nullptr to such a function as an error. 

### Smart Pointers

Guided by the experience of programmers and by solutions provided by the BOOST library, C++11 deprecates auto_ptr and introduces three new smart pointer types: `unique_ptr`, `shared_ptr`, and `weak_ptr`. Chapter 16 discusses the first two.

### Exception Specification Changes

C++ provides a syntax for specifying what exceptions, if any, a function may throw (refer to Chapter 15,“Friends, Exceptions, and More”):

```c++
void f501(int) throw(bad_dog); // can throw type bad_dog exception 
void f733(long long) throw(); // doesn't throw an exception
```

However, the standards committee felt that there is some value in documenting that a function does not throw an exception, and it added the keyword `noexcept` for this purpose:

```c++
void f875(short, short) noexcept; // doesn't throw an exception
```

### Scoped Enumerations

C++11 introduces a variant of enumera- tions that addresses these problems.The new form is indicated by using `class` or `struct` in the definition:

```c++
enum Old1 {yes, no, maybe}; 						// traditional form 
enum class New1 {never, sometimes, often, always}; 	// new form
enum struct New2 {never, lever, sever}; 			// new form
```

### Class Changes

#### `explicit` Conversion Operators

C++ then addressed one aspect of the problem by introducing the keyword `explicit` to suppress automatic conversions invoked by one-argument constructors:

```c++
class Plebe 
{
	Plebe(int); // automatic int-to-plebe conversion 
    explicit Plebe(double); // requires explicit use ...
};
...
Plebe a, b;
a = 5;				// implicit conversion, call Plebe(5)
b = 0.5;			// not allowed
b = Plebe(0.5); 	// explicit conversion
```

C++11 extends the use of `explicit` (discussed in Chapter 11, “Working with Classes”) so that conversion functions can be treated similarly:

```c++
class Plebe
{
...
// conversion functions
    operator int() const;
	explicit operator double() const; 
    ...
};
...
Plebe a, b; 
int n = a; 		// int-to-Plebe automatic conversion
double x = b; 	// not allowed
x = double(b);	// explicit conversion, allowed
```

#### Member In-Class Initialization

Many first-time users of C++ have wondered why they can’t initialize members simply by providing values in the class definition. Now they (and you) can.The syntax looks like this:

```c++
class Session 
{
	int mem1 = 10; // in-class initialization 
    double mem2 {1966.54}; // in-class initialization short mem3;
public: 
    Session(){}														// #1
    Session(short s) : mem3(s) {}									// #2
    Session(int n, double d, short s) : mem1(n), mem2(d), mem3(s) {} // #3
```

### Template and STL Changes

#### Range-based `for` Loop

The loop applies the indicated action to each element in the array or container:

```c++
double prices[5] = {4.99, 10.99, 6.87, 7.99, 8.49}; 
for (auto x : prices)
    std::cout << x << std::endl;
```

If your intent is to have the loop modify elements of the array or container, use a reference type:

```c++
std::vector<int> vi(6);
for (auto & x: vi) // use a reference if loop alters contents
	x = std::rand();
```

#### New STL Containers

C++11 adds `forward_list`, `unordered_map`, `unordered_multimap`, `unordered_set`, and `unordered_multiset` to its collection of STL containers (see Chapter 16).

#### New STL Methods

C++11 adds `cbegin()` and `cend()` to the list of STL methods. Like `begin()` and `end()`, the new methods return iterators to the first element and to one past the last element of a container, thus specifying a range encompassing all the elements. 

In addition, the new methods treat the elements as if they were `const`. Similarly, `crbegin()` and `crend()` are `const` versions of `rbegin()` and `rend()`.

#### `valarray` Upgrade

C++11 adds two functions, `begin()` and `end()`, that each take a `valarray` argument.They return iterators to the first and one past the last element of a `valarray` object, allowing one to use range- based STL algorithms (see Chapter 16).

#### `export` Departs

C++98 introduced the export keyword in the hopes of creating a way to separate tem- plate definitions into interface files containing the prototypes and template declarations and implementation files containing the template function and methods definitions.This proved to be impractical, and C++11 ends that roll for export. However, the Standard retains `export` as a keyword for possible future use.

#### Angle Brackets

To avoid confusion with the `>>` operator, C++ required a space between the brackets in nested template declarations:

```c++
std::vector<std::list<int> > vl; // >> not ok
```

C++11 removes that requirement:

```c++
std::vector<std::list<int>> vl; // >> ok in C++11
```

### The rvalue Reference

Originally, an lvalue was one that could appear on the left side of an assignment statement, but the advent of the `const` modifier allowed for constructs that cannot be assigned to but which are still addressable:

```c++
int n;
int * pt = new int; 
const int b = 101; 		// can't assign to b, but &b is valid
int & rn = n;			// n identifies datum at address &n
int & rt = *pt; 		// *pt identifies datum at address pt
const int & rb = b;		// b identifies const datum at address &b
```

C++11 adds the rvalue reference (discussed in Chapter 8), indicated by using `&&`, that can bind to rvalues—that is, values that can appear on the right-hand side of an assignment expression but for which one cannot apply the address operator. 

```c++
int x = 10;
int y = 23;
int && r1 = 13;
int && r2 = x + y;
double && r3 = std::sqrt(2.0);
```

Note that what `r2` really binds to is the value to which `x + y` evaluates at that time. That is, `r2` binds to the value `23`, and `r2` is unaffected by subsequent changes to `x` or `y`.

```c++
// rvref.cpp -- simple uses of rvalue references
#include <iostream>

inline double f(double tf) {return 5.0*(tf-32.0)/9.0;}; 
int main()
{
    using namespace std;
    double tc = 21.5;
    double && rd1 = 7.07;
    double && rd2 = 1.8 * tc + 32.0; 
    double && rd3 = f(rd2);
    cout << " tc value and address: " << tc <<", " << &tc << endl;
    cout << "rd1 value and address: " << rd1 <<", " << &rd1 << endl;
    cout << "rd2 value and address: " << rd2 <<", " << &rd2 << endl;
    cout << "rd3 value and address: " << rd3 <<", " << &rd3 << endl;
    cin.get();
    return 0; 
}
```

```shell
 tc value and address: 21.5, 0x7ffee5bf0a78
rd1 value and address: 7.07, 0x7ffee5bf0a80
rd2 value and address: 70.7, 0x7ffee5bf0a88
rd3 value and address: 21.5, 0x7ffee5bf0a90

```

## Move Semantics and the Rvalue Reference

### The Need for Move Semantics

```c++
vector<string> allcaps(const vector<string> & vs) 
{
	vector<string> temp;
// code that stores an all-uppercase version of vs in temp
	return temp; 
}
vector<string> vstr;
// build up a vector of 20,000 strings, each of 1000 characters 
vector<string> vstr_copy1(vstr); 			// #1 
vector<string> vstr_copy2(allcaps(vstr)); 	// #2
```



This would be similar to what happens when you move a file from one directory to another:The actual file stays where it is on the hard drive, and just the book- keeping is altered. Such an approach is called **move semantics**. Somewhat paradoxically, move semantics actually avoids moving the primary data; it just adjusts the bookkeeping.

Here’s where the rvalue reference comes into play. We can define two constructors.

One, the regular copy constructor, can use the usual const lvalue reference as a parameter.This reference will bind to lvalue arguments, such as `vstr` in statement #1.

The other, called a **move constructor**, can use an rvalue reference, and it can bind to rvalue arguments, such as the return value of `allcaps(vstr)` in statement #2. 

The copy constructor can do the usual deep copy, while the move constructor can just adjust the bookkeeping.A move constructor may alter its argument in the process of transferring ownership to a new object, and this implies that an rvalue reference parameter should not be `const`.

### A Move Example

```c++
// useless.cpp -- an otherwise useless class with move semantics 
#include <iostream>
using namespace std;
// interface 
class Useless 
{
private:
    int n;          // number of elements
    char * pc;      // pointer to data
    static int ct;  // number of objects
    void ShowObject() const; 
public:
    Useless();
    explicit Useless(int k);
    Useless(int k, char ch);
    Useless(const Useless & f); // regular copy constructor 
    Useless(Useless && f);      // move constructor 
    ~Useless();
    Useless operator+(const Useless & f)const;
// need operator=() in copy and move versions 
    void ShowData() const;
};

// implementation 
int Useless::ct = 0;

Useless::Useless() 
{
    ++ct;
    n = 0;
    pc = nullptr;
    cout << "default constructor called; number of objects: " << ct << endl; 
    ShowObject();
}

Useless::Useless(int k) : n(k) 
{
    ++ct;
    cout << "int constructor called; number of objects: " << ct << endl; 
    pc = new char[n];
    ShowObject();
}

Useless::Useless(int k, char ch) : n(k) 
{
    ++ct;
    cout << "int, char constructor called; number of objects: " << ct
         << endl;
    pc = new char[n];
    for (int i = 0; i < n; i++) 
        pc[i] = ch;
    ShowObject(); 
}

Useless::Useless(const Useless & f): n(f.n) 
{
    ++ct;
    cout << "copy const called; number of objects: " << ct << endl; 
    pc = new char[n];
    for (int i = 0; i < n; i++)
        pc[i] = f.pc[i]; 
    ShowObject();
}

Useless::Useless(Useless && f): n(f.n) 
{
    ++ct;
    cout << "move constructor called; number of objects: " << ct << endl; 
    pc = f.pc;      // steal address
    f.pc = nullptr; // give old object nothing in return
    f.n = 0;
    ShowObject();
}

Useless::~Useless() 
{
    cout << "destructor called; objects left: " << --ct << endl; 
    cout << "deleted object:\n";
    ShowObject();
    delete [] pc;
}

Useless Useless::operator+(const Useless & f)const 
{
    cout << "Entering operator+()\n"; 
    Useless temp = Useless(n + f.n); 
    for (int i = 0; i < n; i++)
        temp.pc[i] = pc[i];
    for (int i = n; i < temp.n; i++)
        temp.pc[i] = f.pc[i - n]; 
    cout << "temp object:\n";
    cout << "Leaving operator+()\n"; 
    return temp;
}

void Useless::ShowObject() const 
{
    cout << "Number of elements: " << n;
    cout << " Data address: " << (void *) pc << endl; 
}

void Useless::ShowData() const 
{
    if (n == 0)
        cout << "(object empty)";
    else
        for (int i = 0; i < n; i++)
            cout << pc[i]; 
    cout << endl;
}

// application 
int main()
{
    {
        Useless one(10, 'x');
        Useless two = one;              // calls copy constructor
        Useless three(20, 'o');
        Useless four (one + three);     // calls operator+(), move constructor
        cout << "object one: ";
        one.ShowData();
        cout << "object two: ";
        two.ShowData();
        cout << "object three: ";
        three.ShowData();
        cout << "object four: "; 
        four.ShowData();
    } 
}
```

```shell
int, char constructor called; number of objects: 1
Number of elements: 10 Data address: 0x7fbee4405940
copy const called; number of objects: 2
Number of elements: 10 Data address: 0x7fbee4405950
int, char constructor called; number of objects: 3
Number of elements: 20 Data address: 0x7fbee4405960
Entering operator+()
int constructor called; number of objects: 4
Number of elements: 30 Data address: 0x7fbee4405980
temp object:
Leaving operator+()
object one: xxxxxxxxxx
object two: xxxxxxxxxx
object three: oooooooooooooooooooo
object four: xxxxxxxxxxoooooooooooooooooooo
destructor called; objects left: 3
deleted object:
Number of elements: 30 Data address: 0x7fbee4405980
destructor called; objects left: 2
deleted object:
Number of elements: 20 Data address: 0x7fbee4405960
destructor called; objects left: 1
deleted object:
Number of elements: 10 Data address: 0x7fbee4405950
destructor called; objects left: 0
deleted object:
Number of elements: 10 Data address: 0x7fbee4405940
```

### Move Constructor Observations

Although using an rvalue reference enables move semantics, it doesn’t magically make it happen.There are two steps to enablement.The first step is that the rvalue reference allows the compiler to identify when move semantics can be used:

```c++
Useless two = one; // matches Useless::Useless(const Useless &) 
Useless four (one + three); // matches Useless::Useless(Useless &&)
```

The object `one` is an lvalue, so it matches the lvalue reference, and the expression
` one + three` is an rvalue, so it matches the rvalue reference.Thus, the rvalue reference directs initialization for object `four` to the move constructor.The second step in enabling move semantics is coding the move constructor so that it provides the behavior we want.

### Assignment

The same considerations that make move semantics appropriate for constructors make them appropriate for assignment. Here, for example, is how you could code the copy assignment and the move assignment operators for the `Useless` class:

```c++
Useless & Useless::operator=(const Useless & f) // copy assignment 
{
	if (this == &f)
        return *this; 
    delete [] pc;
    n = f.n;
    pc = new char[n];
    for (int i = 0; i < n; i++)
    	pc[i] = f.pc[i]; 
    return *this;
}

Useless & Useless::operator=(Useless && f) 		// move assignment
{
    if (this == &f) 
        return *this;
    delete [] pc; 
    n = f.n;
    pc = f.pc;
    f.n = 0;
    f.pc = nullptr; 
    return *this;
}
```

### Forcing a Move

For instance, a program could analyze an array of some sort of candidate objects, select one object for further use, and discard the array. It would be con- venient if you could use a move constructor or a move assignment operator to preserve the selected object. However, suppose you tried the following:

```c++
Useless choices[10];
Useless best;
int pick;
... // select one object, set pick to index 
best = choices[pick];
```

The `choices[pick]` object is an lvalue, so the assignment statement will use the copy assignment operator, not the move assignment operator. But if you could make `choices[pick]` look like an rvalue, then the move assignment operator would be used. This can be done by using the `static_cast<>` operator to cast the object to type `Useless &&`. C++11 provides a simpler way to do this—use the `std::move()` function, which is declared in the `utility` header file. 

```c++
// stdmove.cpp -- using std::move()
#include <iostream>
#include <utility>

// interface 
class Useless 
{
private:
    int n;          // number of elements
    char * pc;      // pointer to data
    static int ct;  // number of objects
    void ShowObject() const; 
public:
    Useless();
    explicit Useless(int k);
    Useless(int k, char ch);
    Useless(const Useless & f); // regular copy constructor 
    Useless(Useless && f);      // move constructor 
    ~Useless();
    Useless operator+(const Useless & f)const;
    Useless & operator=(const Useless & f); // copy assignment
    Useless & operator=(Useless && f);
    void ShowData() const;
};

// implementation 
int Useless::ct = 0;

Useless::Useless() 
{
    ++ct;
    n = 0;
    pc = nullptr;
}

Useless::Useless(int k) : n(k) 
{
    ++ct;
    pc = new char[n];
}

Useless::Useless(int k, char ch) : n(k) 
{
    ++ct;
    pc = new char[n];
    for (int i = 0; i < n; i++) 
        pc[i] = ch;
}

Useless::Useless(const Useless & f): n(f.n) 
{
    ++ct;
    pc = new char[n];
    for (int i = 0; i < n; i++)
        pc[i] = f.pc[i]; 
}

Useless::Useless(Useless && f): n(f.n) 
{
    ++ct;
    pc = f.pc;      // steal address
    f.pc = nullptr; // give old object nothing in return
    f.n = 0;
}

Useless::~Useless() 
{
    delete [] pc;
}

Useless & Useless::operator=(const Useless & f) // copy assignment 
{
	std::cout << "copy assignment operator called:\n";
    if (this == &f)
        return *this; 
    delete [] pc;
    n = f.n;
    pc = new char[n];
    for (int i = 0; i < n; i++)
    	pc[i] = f.pc[i]; 
    return *this;
}

Useless & Useless::operator=(Useless && f) 		// move assignment
{
    std::cout << "move assignment operator called:\n";
    if (this == &f) 
        return *this;
    delete [] pc; 
    n = f.n;
    pc = f.pc;
    f.n = 0;
    f.pc = nullptr; 
    return *this;
}

Useless Useless::operator+(const Useless & f)const 
{
    Useless temp = Useless(n + f.n); 
    for (int i = 0; i < n; i++)
        temp.pc[i] = pc[i];
    for (int i = n; i < temp.n; i++)
        temp.pc[i] = f.pc[i - n]; 
    return temp;
}

void Useless::ShowObject() const 
{
    std::cout << "Number of elements: " << n;
    std::cout << " Data address: " << (void *) pc << std::endl; 
}

void Useless::ShowData() const 
{
    if (n == 0)
        std::cout << "(object empty)";
    else
        for (int i = 0; i < n; i++)
            std::cout << pc[i]; 
    std::cout << std::endl;
}


// application 
int main()
{
    using std::cout; 
    {
        Useless one(10, 'x'); 
        Useless two = one +one;     // calls move constructor
        cout << "object one: "; 
        one.ShowData();
        cout << "object two: "; 
        two.ShowData();
        Useless three, four;
        cout << "three = one\n";
        three = one; // automatic copy assignment 
        cout << "now object three = ";
        three.ShowData();
        cout << "and object one = ";
        one.ShowData();
        cout << "four = one + two\n";
        four = one + two; // automatic move assignment 
        cout << "now object four = ";
        four.ShowData();
        cout << "four = move(one)\n";
        four = std::move(one); // forced move assignment 
        cout << "now object four = ";
        four.ShowData();
        cout << "and object one = ";
        one.ShowData();
    }
}
```

```shell
object one: xxxxxxxxxx
object two: xxxxxxxxxxxxxxxxxxxx
three = one
copy assignment operator called:
now object three = xxxxxxxxxx
and object one = xxxxxxxxxx
four = one + two
move assignment operator called:
now object four = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
four = move(one)
move assignment operator called:
now object four = xxxxxxxxxx
and object one = (object empty)
```

## New Class Features

### Special Member Functions

C++11 adds two more **special member functions** (the move constructor and the move assignment operator) to four previous ones (the default constructor, the copy constructor, the copy assignment operator, and the destructor).These are member functions that the compiler provides automatically, subject to a variety of conditions.

 If the class name is `Someclass`, these two defaulted constructors have the following prototypes:

```c++
Someclass::Someclass(const Someclass &); // defaulted copy constructor 
Someclass::Someclass(Someclass &&); // defaulted move constructor
```

In similar circumstances, the compiler provides a defaulted copy assignment operator and a defaulted move assignment operator with the following prototypes:

```c++
Someclass & Someclass::operator(const Someclass &); // defaulted copy assignment 
Someclass & Someclass::operator(Someclass &&); // defaulted move assignment
```

### Defaulted and Deleted Methods

Suppose that you wish to use a defaulted function that, due to circumstances, isn’t created automatically. For example, if you provide a move constructor, then the default constructor, the copy constructor, and the copy assignment operator are not provided. In that case, you can use the keyword default to explicitly declare the defaulted versions of these methods:

```c++
class Someclass 
{
public:
	Someclass(Someclass &&);
	Someclass() = default; // use compiler-generated default constructor 
    Someclass(const Someclass &) = default;
	Someclass & operator=(const Someclass &) = default;
... 
};
```

The `delete` keyword, on the other hand, can be used to prevent the compiler from using a particular method. For example, to prevent an object from being copied, you can disable the copy constructor and copy assignment operator:

```c++
class Someclass 
{
public:
	Someclass() = default; // use compiler-generated default constructor
// disable copy constructor and copy assignment operator:
    Someclass(const Someclass &) = delete;
    Someclass & operator=(const Someclass &) = delete;
// use compiler-generated move constructor and move assignment operator:
    Someclass(Someclass &&) = default;
    Someclass & operator=(Someclass &&) = default; Someclass & operator+(const Someclass &) const;
... 
};
```

This implies the following:

```c++
Someclass one;
Someclass two;
Someclass three(one); 		// not allowed, one an lvalue 
Someclass four(one + two); 	// allowed, expression is an rvalue
```

**Only the six special member functions can be defaulted**, but you can use `delete` with any member function. One possible use is to disable certain conversions. Suppose, for example, that the `Someclass` class has a method with a type `double` parameter:

```c++
class Someclass 
{
public:
...
	void redo(double); 
...
};
```

Then suppose we have the following code:

```c++
Someclass sc; 
sc.redo(5);
```

The int value `5` will be promoted to `5.0,` and the `redo()` method will execute. 

Now suppose the `Someclass` definition is modified thusly:

```c++
class Someclass 
{
public:
...
    void redo(double);
    void redo(int) = delete;
...
};
```

In this case, the method call `sc.redo(5)` matches the `redo(int)` prototype.The compiler will detect that fact and also detect that `redo(int)` is deleted, and it will then flag the call as a compile-time error.

### Delegating Constructors

To make coding simpler and more reliable, C++11 allows to you use a constructor as part of the definition of another constructor.This process is termed **delegation** because one constructor temporarily delegates responsibility to another constructor to work on the object it is constructing. Delegation uses a variant of the member initialization list syntax:

```c++
class Notes 
{ 
    int k;
	double x;
	std::string st; 
public:
	Notes();
	Notes(int);
	Notes(int, double);
	Notes(int, double, std::string);
};
Notes::Notes(int kk, double xx, std::string stt) : k(kk),
			x(xx), st(stt) {/*do stuff*/}
Notes::Notes() : Notes(0, 0.01, "Oh") {/* do other stuff*/} Notes::Notes(int kk) : Notes(kk, 0.01, "Ah") {/* do yet other stuff*/ } Notes::Notes( int kk, double xx ) : Notes(kk, xx, "Uh") {/* ditto*/ }
```

### Inheriting Constructors

Consider the following code:

```c++
class C1 
{
... 
public: 
...
    int fn(int j) { ... }
	double fn(double w) { ... } 
    void fn(const char * s) { ... }
};
class C2 : public C1 
{
...
public:
...
    using C1::fn;
    double fn(double) { ... }; 
};
...
C2 c2;
int k = c2.fn(3); 		// uses C1::fn(int) 
double z = c2.fn(2.4); 	// uses C2::fn(double)
```

The `using` declaration in `C2` makes the three `fn()` methods in `C1` available to a `C2` object. However, the `fn(double)` method defined in `C2` is chosen over the one from `C1`.

All the constructors of the base class, other than the default, copy, and move constructors, are brought in as possible con-

structors for the derived class, but the ones that have function signatures matching derived class constructors aren’t used:

```c++
class BS
{
    int q;
    double w;
public:
    BS() : q(0), w(0) {}
    BS(int k) : q(k), w(100) {}
    BS(double x) : q(-1), w(x) {}
    B0(int k, double x) : q(k), w(x) {}
    void show() const {std::cout << q << ", " << w << '\n';}
};

class DR : public BS 
{
	short j; 
public:
    using BS::BS;
    DR() : j(-100) {}		// DR needs its own default constructor
    DR(double x) : BS(2*x), j(int(x)) {}
    DR(int i) : j(-2), BS(i, 0.5* i) {}
    void Show() const {std::cout << j << ", "; BS::Show();}
};
int main()
{
    DR o1; 			// use DR()
    DR o2(18.81); 	// use DR(double) instead of BS(double) 
    DR o3(10, 1.8); // use BS(int, double)
    ...
}
```

Because there is no `DR(int, double)` constructor, the inherited `BS(int, double)` is used for `o3`. Note that an inherited base-class constructor only initializes base-class members. If you need to initialize derived class members too, you can use the member list ini- tialization syntax instead of inheritance:

```c++
DR(int i, int k, double x) : j(i), BS(k,x) {}
```

### Managing Virtual Methods: `override` and `final`

For instance, suppose the base class declares a particular virtual method, and you decide to provide a different version for a derived class.This is called **overriding** the old version. But, as discussed in Chapter 13,“Class Inheritance,” if you mismatch the function signature, you hide rather than override the old version:

```c++
class Action 
{
	int a; 
public:
    Action(int i = 0) : a(i) {}
    int val() const {return a;};
    virtual void f(char ch) const { std::cout << val() << ch << "\n";}
};
class Bingo : public Action 
{
public:
	Bingo(int i = 0) : Action(i) {}
	virtual void f(char * ch) const { std::cout << val() << ch << "!\n"; } 
};
```

Because class `Bingo` uses `f(char * ch)` instead of `f(char ch)`, `f(char ch)` is hidden to a `Bingo` object.This prevents a program from using code like the following:

```c++
Bingo b(10);
b.f('@'); // works for Action object, fails for Bingo object
```

With C++11, you can use the virtual specifier `override` to indicate that you intend to override a virtual function. Place it after the parameter list. If your declaration does not match a base method, the compiler objects.Thus, the following version of `Bingo::f()` would generate a compile-time error message:

```c++
virtual void f(char * ch) const override { std::cout << val()
											<< ch << "!\n"; }
```

The specifier `final` addresses a different issue.You may find that you want to prohibit derived classes from overriding a particular virtual method.To do so, place `final` after the parameter list. For example, the following code would prevent classes based on `Action` to redefine the `f()` function:

```c++
virtual void f(char ch) const final { std::cout << val() << ch << "\n";}
```

## Lambda Functions

### The How of Function Pointers, Functors, and Lambdas

Let’s look at an example using three approaches for passing information to an STL algorithm: function pointers, functors, and lambdas.

Suppose you wish to generate a list of random integers and determine how many of them are divisible by 3 and how many are divisible by 13. If necessary, imagine that this is a quest you find absolutely fascinating.

Generating the list is pretty straightforward. One option is to use a `vector<int>` array to hold the numbers and use the STL `generate()` algorithm to stock the array with random numbers:

```c++
#include <vector>
#include <algorithm>
#include <cmath>
...
std::vector<int> numbers(1000); 
std::generate(vector.begin(), vector.end(), std::rand)
```

The `generate()` function takes a range, specified by the first two arguments, and sets each element to the value returned by the third argument, which is a function object that takes no arguments. In this case, the function object is a pointer to the standard `rand()` function.

With the help of the `count_if()` algorithm, it’s easy to count the number of elements divisible by 3.The first two arguments should specify the range, just as for `generate()`. The third argument should be a function object that returns true or false.The `count_if()` function then counts all the elements for which the function object returns `true`. To find elements divisible by 3, you can use this function definition:

```c++
bool f3(int x) {return x % 3 == 0;}
```

With these definitions in place, you can count elements as follows:

```c++
int count3 = std::count_if(numbers.begin(), numbers.end(), f3); 
cout << "Count of numbers divisible by 3: " << count3 << '\n'; 
int count13 = std::count_if(numbers.begin(), numbers.end(), f13); 
cout << "Count of numbers divisible by 13: " << count13 << "\n\n";
```

One advantage of the functor in our example is that you can use the same functor for both counting tasks. Here’s one possible definition:

```c++
class f_mod
{
private:
	int dv; 
public:
	f_mod(int d = 1) : dv(d) {}
	bool operator()(int x) {return x % dv == 0;} 
};
```

Recall how this works.You can use the constructor to create an `f_mod` object storing a particular integer value:

```c++
f_mod obj(3); // f_mod.dv set to 3
```

This object can use the `operator()` method to return a `bool` value:

```c++
bool is_div_by_3 = obj(7); // same as obj.operator()(7)
```

The constructor itself can be used as an argument to functions such as `count_if()`:

```c++
count3 = std::count_if(numbers.begin(), numbers.end(), f_mod(3));
```

The argument `f_mod(3)` creates an object storing the value 3, and `count_if()` uses the created object to call the `operator()()` method, setting the parameter `x` equal to an element of numbers. To count how many numbers are divisible by 13 instead of 3, use `f_mod(13)` as the third argument.

Finally, let’s examine the lambda approach.The name comes from **lambda calculus**, a mathematical system for defining and applying functions.The system enables one to use anonymous functions—that is, it allows one to dispense with function names. 

In the C++11 context, you can use an anonymous function definition (a lambda) as an argument to functions expecting a function pointer or functor.The lambda corresponding to the `f3()` function is this:

```c++
[](int x) {return x % 3 == 0;}
```

It looks much like the definition of `f3()`:

```c++
bool f3(int x) {return x % 3 == 0;}
```

The two differences are that the function name is replaced with `[]` and that there is no declared return type. Instead, the return type is the type that `decltype` would deduce from the return value, which would be bool in this case. If the lambda doesn’t have a return statement, the type is deduced to be void. In our example, you would use this lambda as follows:

```c++
count3 = std::count_if(numbers.begin(), numbers.end(), 
                       [](int x){return x % 3 == 0;});
```

```c++
// lambda0.cpp -- using lambda expressions 
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath> 
#include <ctime> 
const long Size1 = 39L;
const long Size2 = 100*Size1;
const long Size3 = 100*Size2;

bool f3(int x) {return x % 3 == 0;} 
bool f13(int x) {return x % 13 == 0;}

int main() 
{
    using std::cout; 
    std::vector<int> numbers(Size1);

    std::srand(std::time(0));
    std::generate(numbers.begin(), numbers.end(), std::rand);

// using function pointers
    cout << "Sample size = " << Size1 << '\n';

    int count3 = std::count_if(numbers.begin(), numbers.end(), f3); 
    cout << "Count of numbers divisible by 3: " << count3 << '\n'; 
    int count13 = std::count_if(numbers.begin(), numbers.end(), f13); 
    cout << "Count of numbers divisible by 13: " << count13 << "\n\n";

// increase number of numbers
    numbers.resize(Size2);
    std::generate(numbers.begin(), numbers.end(), std::rand); 
    cout << "Sample size = " << Size2 << '\n';
// using a functor 
    class f_mod
    { 
    private:
        int dv; 
    public:
        f_mod(int d = 1) : dv(d) {}
        bool operator()(int x) {return x % dv == 0;} 
    };

    count3 = std::count_if(numbers.begin(), numbers.end(), f_mod(3)); 
    cout << "Count of numbers divisible by 3: " << count3 << '\n'; 
    count13 = std::count_if(numbers.begin(), numbers.end(), f_mod(13)); 
    cout << "Count of numbers divisible by 13: " << count13 << "\n\n";

// increase number of numbers again
    numbers.resize(Size3);
    std::generate(numbers.begin(), numbers.end(), std::rand); 
    cout << "Sample size = " << Size3 << '\n';
// using lambdas
    count3 = std::count_if(numbers.begin(), numbers.end(),
            [](int x){return x % 3 == 0;});
    cout << "Count of numbers divisible by 3: " << count3 << '\n';
    count13 = std::count_if(numbers.begin(), numbers.end(),
            [](int x){return x % 13 == 0;});
    cout << "Count of numbers divisible by 13: " << count13 << '\n';
    
    return 0; 
}
```

```shell
Sample size = 39
Count of numbers divisible by 3: 12
Count of numbers divisible by 13: 0

Sample size = 3900
Count of numbers divisible by 3: 1321
Count of numbers divisible by 13: 314

Sample size = 390000
Count of numbers divisible by 3: 129481
Count of numbers divisible by 13: 30041
```

### The Why of Lambdas

Let’s examine this question in terms of four qualities: proximity, brevity, efficiency, and capability.

Functions are worst because functions cannot be defined inside other functions, so the definition will be located possibly quite far from the point of usage. Functors can be pretty good because a class, including a functor class, can be defined inside a function, so the definition can be located close to the point of use.

In terms of brevity, the functor code is more verbose than the equivalent function or lambda code. Functions and lambdas are approximately equally brief. One apparent exception would be if you had to use a lambda twice:

```c++
count1 = std::count_if(n1.begin(), n1.end(), 
                       [](int x){return x % 3 == 0;});
count2 = std::count_if(n2.begin(), n2.end(), 
                       [](int x){return x % 3 == 0;});
```

But you don’t actually have to write out the lambda twice. Essentially, you can create a name for the anonymous lambda and then use the name twice:

```c++
auto mod3 = [](int x){return x % 3 == 0;} // mod3 a name for the lambda 
count1 = std::count_if(n1.begin(), n1.end(), mod3);
count2 = std::count_if(n2.begin(), n2.end(), mod3);
```

The relative efficiencies of the three approaches boils down to what the compiler chooses to inline. Here, the function pointer approach is handicapped by the fact that compilers traditionally don’t inline a function that has its address taken because the concept of a function address implies a non-inline function.With functors and lambdas, there is no apparent contradiction with inlining.

Finally, lambdas have some additional capabilities. In particular, a lambda can access by name any automatic variable in scope.Variables to be used are **captured** by having their names listed within the brackets. If just the name is used, as in `[z]`, the variable is accessed by value. If the name is preceded by an `&`, as in `[&count]`, the variable is accessed by refer- ence. Using `[&]` provides access to all the automatic variables by reference, and `[=]` pro- vides access to all the automatic variables by value.You also can mix and match. For instance, `[ted, &ed]` would provide access to `ted` by value and `ed` by reference, `[&, ted]` would provide access to `ted` by value and to all other automatic variables by reference, and `[=, &ed]` would provide access by reference to `ed` and by value to the remaining automatic variables. 

```c++
// lambda1.cpp -- use captured variables 
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath>
#include <ctime>

const long Size = 390000L;

int main() 
{
    using std::cout; 
    std::vector<int> numbers(Size);
    
    std::srand(std::time(0));
    std::generate(numbers.begin(), numbers.end(), std::rand); 
    cout << "Sample size = " << Size << '\n';
// using lambdas
    int count3 = std::count_if(numbers.begin(), numbers.end(),
        [](int x){return x % 3 == 0;});
    cout << "Count of numbers divisible by 3: " << count3 << '\n'; 
    int count13 = 0;
    std::for_each(numbers.begin(), numbers.end(),
        [&count13](int x){count13 += x % 13 == 0;});
    cout << "Count of numbers divisible by 13: " << count13 << '\n';
// using a single lambda
    count3 = count13 = 0; std::for_each(numbers.begin(), numbers.end(),
        [&](int x){count3 += x % 3 == 0; 
    count13 += x % 13 == 0;}); 
    cout << "Count of numbers divisible by 3: " << count3 << '\n'; 
    cout << "Count of numbers divisible by 13: " << count13 << '\n'; 
    return 0;
}
```

```shell
Sample size = 390000
Count of numbers divisible by 3: 130185
Count of numbers divisible by 13: 29957
Count of numbers divisible by 3: 130185
Count of numbers divisible by 13: 29957
```

## Wrappers

### The `function` Wrapper and Template Inefficiencies

```c++
// somedefs.h
#include <iostream>

template <typename T, typename F> 
T use_f(T v, F f)
{
    static int count = 0; 
    count++;
    std::cout << " use_f count = " << count
              << ", &count = " << &count << std::endl;
    return f(v);
}

class Fp 
{ 
private:
    double z_; 
public:
    Fp(double z = 1.0) : z_(z) {} 
    double operator()(double p) { return z_*p; }
};

class Fq 
{ 
private:
    double z_; 
public:
    Fq(double z = 1.0) : z_(z) {} 
    double operator()(double q) { return z_+ q; }
};
```

```c++
// callable.cpp -- callable types and templates 
#include "somedefs.h"
#include <iostream>

double dub(double x) {return 2.0*x;} 
double square(double x) {return x*x;}

int main() 
{
    using std::cout; 
    using std::endl;

    double y = 1.21;
    cout << "Function pointer dub:\n";
    cout << " " << use_f(y, dub) << endl;
    cout << "Function pointer square:\n";
    cout << " " << use_f(y, square) << endl;
    cout << "Function object Fp:\n";
    cout << " " << use_f(y, Fp(5.0)) << endl;
    cout << "Function object Fq:\n";
    cout << " " << use_f(y, Fq(5.0)) << endl;
    cout << "Lambda expression 1:\n";
    cout << " " << use_f(y, [](double u) {return u*u;}) << endl; 
    cout << "Lambda expression 2:\n";
    cout << " " << use_f(y, [](double u) {return u+u/2.0;}) << endl; 
    return 0;
}
```

```shell
Function pointer dub:
  use_f count = 1, &count = 0x109cbd080
2.42
Function pointer square:
  use_f count = 2, &count = 0x109cbd080
1.4641
Function object Fp:
  use_f count = 1, &count = 0x109cbd084
6.05
Function object Fq:
  use_f count = 1, &count = 0x109cbd088
6.21
Lambda expression 1:
  use_f count = 1, &count = 0x109cbd08c
1.4641
Lambda expression 2:
  use_f count = 1, &count = 0x109cbd090
1.815
```

The template function `use_f()` has a static member `count`, and we can use its address to see how many instantiations are made.There are five distinct addresses, so there must have been five distinct instantiations of the `use_f()` template.

 First, look at this call:

```c++
use_f(y, dub);
```

Here `dub` is the name of a function that takes a `double` argument and returns a `double` value. The name of a function is a pointer, hence the parameter `F` becomes type `double (*)(double)`, a pointer to a function with a `double` argument and a `double` return value.

The next call is this:

```c++
use_f(y, square);
```

Again, the second argument is type `double (*)(double)`, so this call uses the same instantiation of `use_f()` as the first call.

The next two calls to `use_f()` have objects as second arguments, so `F` becomes type `Fp` and `Fq` respectively, so we get two new instantiations for these values of `F`. Finally, the last two calls set `F` to whatever types the compiler uses for lambda expressions.

### Fixing the Problem

The function wrapper lets you rewrite the program so that it uses just one instantiation of `use_f()` instead of five. 

The `function` template, declared in the `functional` header file, specifies an object in terms of a call signature, and it can be used to wrap a function pointer, function object, or lambda expression having the same call signature. For example, the following declaration creates a `function` object `fdci` that takes a `char` and an `int` argument and returns type `double`:

```c++
std::function<double(char, int)> fdci;
```

You can then assign to `fdci` any function pointer, function object, or lambda expres- sion that takes type `char` and `int` arguments and returns type `double`.

```c++
//wrapped.cpp -- using a function wrapper as an argument 
#include "somedefs.h"
#include <iostream>
#include <functional>

double dub(double x) {return 2.0*x;} 
double square(double x) {return x*x;}

int main() 
{
    using std::cout; 
    using std::endl;
    using std::function;

    double y = 1.21;
    function<double(double)> ef1 = dub;
    function<double(double)> ef2 = square; 
    function<double(double)> ef3 = Fq(10.0); 
    function<double(double)> ef4 = Fp(10.0); 
    function<double(double)> ef5 = [](double u) {return u*u;}; 
    function<double(double)> ef6 = [](double u) {return u+u/2.0;};
    cout << "Function pointer dub:\n";
    cout << " " << use_f(y, ef1) << endl;
    cout << "Function pointer square:\n";
    cout << " " << use_f(y, ef2) << endl;
    cout << "Function object Fp:\n";
    cout << " " << use_f(y, ef3) << endl;
    cout << "Function object Fq:\n";
    cout << " " << use_f(y, ef4) << endl;
    cout << "Lambda expression 1:\n";
    cout << " " << use_f(y, ef5) << endl; 
    cout << "Lambda expression 2:\n";
    cout << " " << use_f(y, ef6) << endl; 
    return 0;
}
```

```shell
Function pointer dub:
  use_f count = 1, &count = 0x102b693f0
2.42
Function pointer square:
  use_f count = 2, &count = 0x102b693f0
1.4641
Function object Fp:
  use_f count = 3, &count = 0x102b693f0
11.21
Function object Fq:
  use_f count = 4, &count = 0x102b693f0
12.1
Lambda expression 1:
  use_f count = 5, &count = 0x102b693f0
1.4641
Lambda expression 2:
  use_f count = 6, &count = 0x102b693f0
1.815
```

### Further Options

First, we don’t actually have to declare six `function<double(double)>` objects in Listing 18.8. Instead, we can use a temporary `function<double(double)>` object as an argument to the `use_f()` function:

```c++
typedef function<double(double)> fdd; // simplify the type declaration
cout << use_f(y, fdd(dub)) << endl; // create and initialize object to dub 
cout << use_f(y, fdd(square)) << endl;
...
```

Second, Listing 18.8 adapts the second arguments in `use_f()` to match the formal parameter `f`. Another approach is to adapt the type of the formal parameter `f` to match the original arguments.This can be done by using a function wrapper object as the second parameter for the `use_f()` template definition.We can define `use_f()` this way:

```c++
#include <functional>
template <typename T>
T use_f(T v, std::function<T(T)> f) // f call signature is T(T)
{
    static int count = 0;
    count++;
    std::cout << " use_f count = " << count
    		  << ", &count = " << &count << std::endl; 
    return f(v);
}
```

Then the function calls can look like this:

```c++
cout << " " << use_f<double>(y, dub) << endl;
...
cout << " " << use_f<double>(y, Fp(5.0)) << endl;
...
cout << " " << use_f<double>(y, [](double u) {return u*u;}) << endl;
```

## Variadic Templates

Variadic templates provide a means to create template functions and template classes that accept a variable number of arguments. For instance, consider this code:

```c++
int n = 14;
double x = 2.71828;
std::string mr = "Mr. String objects!"; 
show_list(n, x);
show_list(x*x, '!', 7, mr);
```

The goal is to be able to define `show_list()` in such a way that this code would compile and lead to this output:

```c++
14, 2.71828
7.38905, !, 7, Mr. String objects!
```

There are a few key points to understand in order to create variadic templates:

* Template parameter packs 
* Function parameter packs 
* Unpacking a pack

* Recursion

### Template and Function Parameter Packs

As a starting point to see how parameter packs work, let’s consider a simple template function, one that displays a list consisting of just one item:

```c++
template<typename T> void show_list0(T value) 
{
	std::cout << value << ", "; 
}
```

This definition has two parameter lists.The template parameter list is just `T`. The function parameter list is just `value`. A function call such as the following sets `T` in the template parameter list to `double` and `value` in the function parameter list to `2.15`:

```c++
show_list0(2.15);
```

C++11 provides an ellipsis meta-operator that enables you to declare an identifier for a template parameter pack, essentially a list of types. 

```c++
template<typename... Args> 		// Args is a template parameter pack 
void show_list1(Args... args) 	// args is a function parameter pack 
{
...
}
```

`Args` is a template parameter pack, and `args` is a function parameter pack. The difference between Args and T is that T matches a single type, whereas Args matches any number of types, including none. Consider the following function call:

```c++
show_list1('S', 80, "sweet", 4.5);
```

In this case the parameter pack `Args` contains the types matching the parameters in the function call: `char`, `int`, `const char *`, and `double`.

Next, much as

```c++
void show_list0(T value)
```

states that value is of type `T`, the line

```c++
void show_list1(Args... args) // args is a function parameter pack
```

states that `args` is of type `Args`.  More precisely, this means that the function pack args contains a list of values that matches the list of types in the template pack Args, both in type and in number.

### Unpacking the Packs

You can unpack the pack by placing the ellipsis to the right of the function parameter pack name. For example, consider the following flawed code:

```c++
template<typename... Args> // Args is a template parameter pack 
void show_list1(Args... args) // args is a function parameter pack 
{
	show_list1(args...); // passes unpacked args to show_list1() 
}
```

What does this mean, and why is it flawed? Suppose we have this function call:

```c++
show_list1(5,'L',0.5);
```

The call packs the values `5`, `'L'`, and `0.5` into `args`. Within the function, the call

```c++
show_list1(args...);
```

expands to the following:

```c++
show_list1(5,'L',0.5);
```

### Using Recursion in Variadic Template Functions

``` c++
//variadic1.cpp -- using recursion to unpack a parameter pack 
#include <iostream>
#include <string>

// definition for 0 parameters -- terminating call
void show_list3() {}

// definition for 1 or more parameters 
template<typename T, typename... Args> 
void show_list3( T value, Args... args) 
{
    std::cout << value << ", ";
    show_list3(args...); 
}
int main() 
{
    int n = 14;
    double x = 2.71828;
    std::string mr = "Mr. String objects!"; 
    show_list3(n, x);
    show_list3(x*x, '!', 7, mr);
    return 0;
}
```

```shell
14, 2.71828, 7.38905, !, 7, Mr. String objects!, % 
```

Consider this function call:

```c++
show_list3(x*x, '!', 7, mr);
```

The first argument matches `T` to `double` and `value` to `x*x`. The remaining three types (`char`, `int`, and `std::string`) are placed in the `Args` pack, and the remaining three values (`'!'`, `7`, and `mr`) are placed in the `args` pack.

Next, the `show_list3()` function uses `cout` to display `value` (approximately `7.38905`) and the string `", "`.That takes care of displaying the first item in the list.

Next comes this call:

```c++
show_list3(args...);
```

This, given the expansion of `args...`, is the same as the following:

```c++
show_list3('!', 7, mr);
```

```c++
// variadic2.cpp 
#include <iostream> 
#include <string>

// definition for 0 parameters 
void show_list() {}

// definition for 1 parameter 
template<typename T>
void show_list(const T& value) 
{
    std::cout << value << '\n'; 
}

// definition for 2 or more parameters 
template<typename T, typename... Args>
void show_list(const T& value, const Args&... args) 
{
    std::cout << value << ", ";
    show_list(args...); 
}

int main() 
{
    int n = 14;
    double x = 2.71828;
    std::string mr = "Mr. String objects!"; 
    show_list(n, x);
    show_list(x*x, '!', 7, mr);
    return 0;
}
```

```shell
14, 2.71828
7.38905, !, 7, Mr. String objects!
```

## More C++11 Features

### Concurrent Programming

C++11 addresses concurrency by defining a memory model that supports threaded execution, by adding the keyword `thread_local`, and by providing library support.The keyword `thread_local` is used to declare variables having static storage duration relative to a particular thread; that is, they expire when the thread in which they are defined expires.

The library support consists of the atomic operations library, which specifies the `atomic` header file, and the thread support library, which specifies the `thread`, `mutex`, `condition_variable`, and `future` header files.

### Library Additions

C++11 adds several specialized libraries. An extensible random number library, supported by the `random` header file, provides much more extensive and sophisticated random number facilities than `rand()`. For example, it offers a choice of random-number generators and a choice of distributions, including uniform (like `rand()`), binomial, normal, and several others.

The `chrono` header file supports ways to deal with time duration.

The `tuple` header file supports the `tuple` template. A `tuple` object is a generalization of a pair object.Whereas a pair object can hold two values whose types need not be the same, a tuple can hold an arbitrary number of items of different types.

The compile-time rational arithmetic library, supported by the `ratio` header file, allows the exact representation of any rational number whose numerator and denomina- tor can be represented by the widest integer type. It also provides arithmetic operations for these numbers.

One of the most interesting additions is a regular expression library, supported by the `regex` header file. A regular expression specifies a pattern that can be used to match con- tents in a text string. For example, a bracket expression matches any single character in the brackets. Thus, `[cCkK]` matches a single `c`, `C`, `k`, or `K`, and `[cCkK]`at matches the words `cat`, `Cat`, `kat`, and `Kat`.

### Low-Level Programming

Low level means closer to the bits and bytes of computer hardware and machine language. Low-level programming is important for embedded pro- gramming and for increasing the efficiency of some operations. C++11 offers some aids to those who do low-level programming.

One change is relaxing the constraints on what qualifies as “Plain Old Data,” or POD.

Another change is making unions more flexible by allowing them to have members that have constructors and destructors, but with some restrictions on other properties, for example, not allowing virtual functions. 

### Miscellaneous

C++11 follows the lead of C99 in allowing for implementation-dependent extended integer types. Such types, for example, could be used on a system with 128-bit integers. Extended types are supported in the C header file `stdint.h` and in the C++ version, `cstdint`.

C++11 provides a mechanism, the **literal operator**, for creating user-defined literals. Using this mechanism, for instance, one can define binary literals, such as `1001001b`, which the corresponding literal operator will convert to an integer value.

C++ has a debugging tool called `assert`. It is a macro that checks during runtime if an assertion is true and which displays a message and calls `abort()` if the assertion is false. The assertion would typically be about something the programmer thinks should be true at that point in the program. C++11 adds the keyword `static_assert`, which can be used to test assertions during compile time.The primary motivation is to make it easier to debug templates for which instantiation takes place during compile time, not runtime.

C++11 provides more support for metaprogramming, which is creating programs that create or modify other programs or even themselves. In C++ this can be done during compile time using templates.

## Language Change

### The Boost Project

The Boost project began in 1998 when Beman Dawes, the then-chairman of the C++ library working group, brought together a few other members of the group and developed a plan to generate new libraries outside the confines of the standards committee.The basic idea was to provide a website that acts as an open forum for people to post free C++ libraries.

Boost has over 100 libraries at the time of this writing, and they can be downloaded as a set from [boost](www.boost.org), as can documentation. Most of the libraries can be used by including the appropriate header files.

### The TR1

The Technical Report 1, or TR1, was a project of a subset of the C++ Standards committee.TR1 was a compilation of library extensions that were compatible with the C++98 standard, but which were not required by the standard.They were candidates for the next iteration of the standard.

### Using Boost

```c++
// lexcast.cpp -- simple cast from float to string 
#include <iostream>
#include <string>
#include "boost/lexical_cast.hpp"

int main() 
{
    using namespace std;
    cout << "Enter your weight: ";
    float weight;
    cin >> weight;
    string gain = "A 10% increase raises ";
    string wt = boost::lexical_cast<string>(weight);
    gain = gain + wt + " to "; // string operator+() 
    weight = 1.1 * weight;
    gain = gain + boost::lexical_cast<string>(weight) + "."; 
    cout << gain << endl;
    return 0;
}
```

`lexical_cast` from the Conversion library provides simple conversions between numeric and string types.The syntax is modeled after `dynamic_cast`, in which you provide the target type as a template parameter.

```c++
Enter your weight: 150
A 10% increase raises 150 to 165.
```