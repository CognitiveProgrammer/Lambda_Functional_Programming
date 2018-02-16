# Functional Programming in C++ Using Lambda : Detailed Usage Guide

Lambdas are one of the totally new addition to C++11 and has changed the way we write the code in C++. It allows creation of in-place anonymous functions and enables C++ to be a full fledged __Functional Programming__ Language.

Since the syntax as well as usage is relatively new, many C++ programmers still find it bit difficult to write and the usage scenarios. This tutorial intends to describe the syntax, usage, and applicabilities of lambdas.

## Topic- 0: An introduction to Functional Programming

In a nutshell, __Functional Programming__ is all about treating the code in a way we treat functions in mathematics. In mathematics, functions are only dependent upon the input parameters. With a given input, the function will always the same output.

We can consider some of the general mathematical functions like

```
f(x)  = x * x

```
In here, no matter what the conditions are `f(10)` will always generate `100` and `f(5)` will always generate `25.`

and

```
f(x,y)2 = x2 + y2 + 2xy
```
Here, `f(3,1)` will always generate `16` and `f(5,2)` will always generate `49`. In other words, these functions will not have any side effects because they are neither dependent on nor change any external state outside of the function

In __Functional Programming__, we create functions similar to the way we have written a couple of mathematical functions above and lambdas enables us to do the same.

### Topic- 0.1: The emergence of Functional Programming

__Functional Programming__ is not new but gaining attractions today because of the large scale availability of multi core CPU(s). Functions without any side can easily run on multiple CPU(s) simultaneously without using extensive synchronization mechanisms

_Note: It was possible to write functional programs without lambdas, just that it is difficult to write as well as manage the code_


## Topic- 1: Lambda Syntax and Calling Mechanism

The syntax of lambda consists of `[]`, `()`, and `{}`. This is a valid lambda syntax and shall be successfully compiled with the compilers

```
[](){};

```
- `[]` is Capture list
- `()` is Function argument provider
- `{}` is lambda implementation

we can create a very simple lambda which just prints hello world as

```
[](){
       cout<<"Hello World"<<endl;
   };
```
However, the lambda can't call itself automatically, we need to call them explicitly. We can call the lambda in-place in a similar way we call any other normal functions as

```
[](){
    cout<<"Hello World"<<endl;
}();
```
Lambdas as a function can also return values and the return type of a lambda is depicted using `->` syntax as
```
[]() -> int{
    cout<<"Hello World"<<endl;
    return 1;
}();
```
We can collect the return value in any integer as
```
int retLambda = []()->int{
    cout<<"Hello World"<<endl;
    return 1;
}();
```
Just like in a normal function, we can also pass the parameters in the lambda function inside `()` as
```
int retLambda = [](int value)->int{
   cout<<"Hello World"<<endl;
   return value + 1;
}(100);
```

The lambda functions need not be called in-place, we can very well keep the lambda inside a function pointer and call it at a place of our choice. We can use C++ `auto` keyword, or use `std::function<>` template to hold the function pointers, we can also use `C` style function pointers for the same, but is generally avoided because of complicated syntax.

```
std::function<int(int)> lambdaFn = [](int value)->int{
    cout<<"Hello World"<<endl;
    return value + 1;
};
int retLambda = lambdaFn(100);
```

The lambda function is just like the mathematical function where it will always return `101` when the value `100` is passed on to it.

## Topic- 2: Lambda functions as Mathematical functions

For the function `f(x) = x * x` we can write the lambda functions as

```
std::function<int(int)> fxsquare = [](int x) ->int {
    return x * x;
};
int retValue = fxsquare(10);
```
It this doesn't look like a big deal, let's try to write C++ lambda for `f(x,y)2 = x2 + y2 + 2xy`

```
std::function<int(int, int)> fxsquare = [](int x, int y) ->int {
    int xsquare = [](int x) { return x * x; }(x);
    int ysquare = [](int y) { return y * y; }(y);
    int twoxy = [](int x, int y) { return 2 * x * y;}(x,y);
    return xsquare + ysquare + twoxy;
};
int retValue = fxsquare(5,3);

```
Here we have created 3 lambda functions inside a lambda function to calculate the `3` different values which is finally being summed and returned. Some of the obvious benefits of using lambda functions for doing this is

- We're not creating multiple functions in global scope which will stay forever. All lambda functions inside are not available outside and shall be out of scope as soon as the function is executed.
- Given an input, all lambda functions will return the same output i.e No side effects

## Topic- 3: Using Capture list

The `[]` is called capture list and is one of the most interesting aspect of C++ lambda. A lambda function can be defined in-place inside a function but, it remains an independent function and doesn't have access to the local scope variables.

The capture list `[]` is used to make local scope variables available to the lambda functions without passing them explicitly inside the parameter argument lists.

The local scope variables can be made available either by value (i.e a copy) or by reference. To pass a value, we need to specify local scope variables in the capture list. Only variables mentioned in the capture list shall be made available to the lambda function

```
int localvar = 100;
[localvar](){
    cout<<"Localvar =>"<<localvar<<endl;
}();
```
Pass by value means has a special meaning in lambda

- All pass by value parameters are immutable. i.e we can't change the values of the variables.

For example, the code below shall generate a compiler error
```
int localvar = 100;
[localvar](){
    localvar++; // Compiler errror
}();
```
We can use `&` to pass the value by reference inside the lambda. In this case, we can change the values inside lambda which will be reflected outside of the lambda. In short, the existing reference (i.e`&`) mechanism of C++ holds good here.

```
int localvar = 100;
  [&localvar](){
      localvar++;
}();
cout<<"Localvar =>"<<localvar<<endl;
```
We can not only specify multiple variables in the capture list, but can also mix and match pass by value and pass by reference.

```
int localvar = 100;
int localvar2 = 200;
[&localvar, localvar2](){
   localvar++; // Compiler Error
   localvar2++; // Okay
}();
```

### Topic- 3.1: Using Capture list - Multiple Parameters

We know that we can specify multiple parameters in capture list, however there is a shortcut way of passing all local scope variables inside the lambda. We can use `=` in capture list to pass all by value and `&` to pass all by reference.

```
int localvar = 100;
int localvar2 = 200;
[=](){
   localvar++; // Passed by Value.. immutable.. compiler error
   localvar2++;
}();
```
and
```
int localvar = 100;
int localvar2 = 200;
[&](){
    localvar++; // Passed all by reference.. fine
    localvar2++;
}();

```
We can also selectively override the value of individual variable. For example, if we want to pass all parameters by value, except one which needs to be passed by reference, we can use

```
int localvar = 100;
int localvar2 = 200;
[=,&localvar2](){
    localvar++; // Compiler Error
    localvar2++; // Fine
}();
```

## Topic- 4: Using Capture list in the existing function

Let's go back to the function `fxsquare` we have created earlier.
```
std::function<int(int, int)> fxsquare = [](int x, int y) ->int {
        int xsquare = [](int x) { return x * x; }(x);
        int ysquare = [](int y) { return y * y; }(y);
        int twoxy = [](int x, int y) { return 2 * x * y;}(x,y);
        return xsquare + ysquare + twoxy;
    };
```
In here, we're passing the parameters `x & y` into the function. We can take these parameters in the capture list as
```
int x = 5, y = 3;
std::function<int(void)> fxsquare = [x,y]() ->int {
    int xsquare = [](int x) { return x * x; }(x);
    int ysquare = [](int y) { return y * y; }(y);
    int twoxy = [](int x, int y) { return 2 * x * y;}(x,y);
    return xsquare + ysquare + twoxy;
};
int retValue = fxsquare();
```
or
```
int x = 5, y = 3;
std::function<int(void)> fxsquare = [=]() ->int {
    int xsquare = [](int x) { return x * x; }(x);
    int ysquare = [](int y) { return y * y; }(y);
    int twoxy = [](int x, int y) { return 2 * x * y;}(x,y);
    return xsquare + ysquare + twoxy;
};
int retValue = fxsquare();
```
We can even put everything inside a function and return a lambda function from the function as

```
auto retFxSquare() {
  int x = 5, y = 3;
  return [x,y]() ->int {
      int xsquare = [](int x) { return x * x; }(x);
      int ysquare = [](int y) { return y * y; }(y);
      int twoxy = [](int x, int y) { return 2 * x * y;}(x,y);
      return xsquare + ysquare + twoxy;
  };
}
void main() {
  auto fxsquare = retFxSquare();
  int retValue = fxsquare();
}
```
In this case, we can even access the local variables `x & y` of the function `retFxSquare` because of the concept called __Closures__, where it copies the variable when the function is returned as a function from the function `retFxSquare`


## Topic- 5: Using Capture list with classes

When we pass anything in the capture list by value, a copy is created and copy constructor will be called for the class.

```
struct MyClass {
    MyClass(const MyClass &) { cout<<"MyClass Copy Constructor..."<<endl;}
};

MyClass mclass;
[mclass](){};

```

### Topic- 5.1: Using Capture list - for class variables

We can use class variables in capture list by passing `this` because class variable are not available locally inside a class function but `this` is available.

```
template<int iValue, int jValue>
struct AbcTest {
  int i = iValue;
  int j = jValue;
  int sumFn() {
      return [this](){
          return this->i + this->j;
      }();
  }
};

AbcTest<100,200> aTest;
cout<<aTest.sumFn()<<endl;
```

## Conclusion

We've seen what is functional programming, lambdas and how to use the same within as well as outside of the classes.

