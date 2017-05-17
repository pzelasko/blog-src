Title: Yet another fizzbuzz...
Lead: this time, in compile time.
Published: 7/5/2017
Tags: 
    - C++
    - template metaprogramming
---
By now, everybody has to know the infamous fizzbuzz programming interview task. Is it a good test of one's coding skills and problem-solving attitude? Honestly, I don't know. What I do know, is that some poeple are having great fun mocking the excercise (yes, myself included). My favourite so far is [Joel's take on tensorflow fizzbuzz solution](http://joelgrus.com/2016/05/23/fizz-buzz-in-tensorflow/).

So, if a neural network can solve fizzubuzz, why shouldn't we task a C++ compiler with the same, right? Sure we can! I'll try to explain how it works.

TL;DR:

    #include <iostream>
    
    template<bool mod_3, bool mod_5, unsigned number>
    struct fizzbuzz { enum { value = number }; };
     
    template<unsigned number>
    struct fizzbuzz<false, false, number> { static const char value[]; };
    template<unsigned number> const char fizzbuzz<false, false, number>::value[] = "fizzbuzz"; 
     
    template<unsigned number>
    struct fizzbuzz<false, true, number> { static const char value[]; };
    template<unsigned number> const char fizzbuzz<false, true, number>::value[] = "fizz"; 
     
    template<unsigned number>
    struct fizzbuzz<true, false, number> { static const char value[]; };
    template<unsigned number> const char fizzbuzz<true, false, number>::value[] = "buzz"; 
     
    template <unsigned idx>
    struct iteration {
        iteration<idx + 1> next;
        void operator()() const { 
            std::cout << fizzbuzz<static_cast<bool>(idx % 3), static_cast<bool>(idx % 5), idx>::value << std::endl;
            next();
        }
    };
     
    template<>
    struct iteration<100> {
        void operator()() const {}
    };
     
    int main() {
        iteration<1>{}();
    }

By now, you might wonder, "what kind of craziness is this stuff?!" Well, let me explain myself then!

Firstly, the assumption is to move as much of the program logic from runtime to compile time. What it means, for example, is that instead of generating code which checks whether a number is divisible by 3, I want the compiler to generate code which already checked that during compilation, and only prints to the console the right answer for a given number. Why would I do that? For fun of course! However, in reality, it turns out that it's actually quite a useful technique when performing some kinds of optimizations, since you're eliminating branching from your runtime code (however, at the cost of compilation time and possible generated code bloat) [also, while we're at it, remember, that [premature optimization is the root of all evil!](https://en.wikiquote.org/wiki/Donald_Knuth)].

So where do we start? Normally when iterating over one hundred numbers, one would probably use a for/while loop. No such thing here! We want our iteration to be performed in compile time. It seems like we could use just generate 100 different objects, each printing the right thing to the console, and call them one after another. One way to do this is to use recursion. But how do we generate those objects? Well, template metaprogramming comes to the rescue!

    template <unsigned idx>
    struct iteration {
        iteration<idx + 1> next;
        void operator()() const {
            /* print the right thing */
            next();
        }
    };
     
    template<>
    struct iteration<100> {
        void operator()() const {}
    }; 

We first define a struct named iteration. Since we templated the structure on an unsigned int we named index, we're in fact creating about 100 classes, each of which can use its individual index to determine what should be written to standard output, and can call the iteration after it (for that, we make the objects of these classes callable using the operator()). Should we only define the general recipe for iteration, it would recurse infinitely, and the compiler would stop at some point and yell at us. That's good! The compiler should yell at us for doing such silly mistakes. So, in order to terminate the recursion, we create a template specialization for the struct iteration at index 100, which doesn't point at another iteration and simply does nothing.

Okay, so we've successfully replaced the loop. Now, we've got to determine when to write "fizz", "buzz", "fizzbuzz", or the number. That's the part where you usually write a series of ifs and elses. Not gonna happen here!

    template<bool mod_3, bool mod_5, unsigned number>
    struct fizzbuzz { enum { value = number }; };
     
    template<unsigned number>
    struct fizzbuzz<false, false, number> { static const char value[]; };
    template<unsigned number> const char fizzbuzz<false, false, number>::value[] = "fizzbuzz";
     
    template<unsigned number>
    struct fizzbuzz<false, true, number> { static const char value[]; };
    template<unsigned number> const char fizzbuzz<false, true, number>::value[] = "fizz";
     
    template<unsigned number>
    struct fizzbuzz<true, false, number> { static const char value[]; };
    template<unsigned number> const char fizzbuzz<true, false, number>::value[] = "buzz";

Instead of writing conditional branches, we will make the compiler select the right branch at compile time. Since we already know all the numbers (1-100) at compile time, we observe that we can also determine whether they are divisible by 3 and by 5 at compile time. Conditional statements are now replaced by a templated struct fizzbuzz, which by default allows to access the number by using an enum with a constant named value defined inside. After having written the general case, we can specialize this template to write proper output depending on conditions being satisfied - e.g. when `number % 3` evaluates to 0, and `number % 5` evaluates to something other than 0, we cast them to bool (resulting in false and true) - we make this specialization have a `const char[]` member `value` with value "fizz".

Since we have all specializations defined, we will get either an `int` (more specifically, enum) or a `const char[]` when evaluating
`fizzbuzz<static_cast<bool>(idx % 3), static_cast<bool>(idx % 5), idx>::value`.
We can just pass that to std::cout.

Now, all we have to do, is call first iteration in main.

That's all there is to it. If you don't believe the code doesn't have any loops or conditional branches, [have a look at assembly at coliru](http://coliru.stacked-crooked.com/a/3ce6013f2223f7e7). 

I hope you enjoyed the post. Please let me know about that in the comments. Till' next time!