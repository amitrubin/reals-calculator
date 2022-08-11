![](https://raw.githubusercontent.com/amitrubin/R-Arithmetic/main/R%20Arithmetic%20Logo2.png)

This JavaScript library provides Arbitrarily Precise Calculations over real computable numbers.  

E.g. give me the first 100 decimal digits of 3*e/pi: '2.5957679382967952616533243689382688522862338717255466183629908668441351337120311758957739875485535519'  
## How to set up the library

Simply download the two files (*fractions.js*, *continued_fractions.js*). Make sure you put them in the same directory.

## The Basics (Structure and Theory)

The calculator is made up of two components. 

The first is a straight-forward fractions (rational numbers) calculator. It is implemented in the file *fractions.js*  

However, in this project I sought (under the guidance of professor Mayer Goldberg) to to include irrational numbers such as the square root of two, pi, cosh(1.234567), etc. Importantly, the results should always be as precisely accurate as we desire.

For this purpose we utilized infinite [Continued Fractions](https://en.wikipedia.org/wiki/Continued_fraction) as foundation. There is good reason to do so: continued fractions have great convergence properties, and many irrational numbers have known continued fraction identities. These are implemented in *continued_fractions.js*  

However, we're still missing a crucial component - precise arithmetic between the irrationals, and between irrationals and rationals! For this I utilized [Bill Gosper's 1972 algorithms](https://perl.plover.com/yak/cftalk/INFO/gosper.txt), though with a great deal of modifications. This arithmetic is also included in *continued_fractions.js*

## Guide

### Example code 1: getting an approximation.

Let's find the first 100 digits of pi & of arctan(0.9):  

    let cf = require("./continued_fractions.js");  
    let fr = require("./fractions.js");  

    let a = cf.Constants.pi(); // the constant pi, i.e. 3.14159...  

    let ninth = fr.Fraction.make_fraction_from_string("9/10")`; // you could also write fr.Fraction.make_fraction_from_bignums(9n,10n);  
    let b = cf.Constants.arctan(ninth);

    let str1 = a.to_decimal_string(100);    
    let str2 = b.to_decimal_string(100);  
    console.log("pi: " + str1);  
    console.log("arctan(0.9): " + str2);  

Now let's break that down:
 
#### *Importing the library*

Step one is importing *continued_fractions.js* and *fractions.js*.  

    let cf = require("./continued_fractions.js");  
    let fr = require("./fractions.js");  

Next, we need two classes from cf. 

There are the irrational constants: `cf.Constants;`  
and there are factory methods for the rest: `cf.CF;`  

(there is currently some overlap between the two classes. This will be corrected soon - if this notice is still around in 2023 please nudge me to fix it)

#### *Constants*

The Constants class has factory methods that return new copies of mathematical constants. A full list of available methods is available in appendix 1 below, or by running console.log(cf.Constants.toString()).  

Here are two examples, with and without parameters. First, pi:

    let a = cf.Constants.pi(); // the constant pi, i.e. 3.14159...

Easy enough. But what if we require a parameter, i.e. arctan(0.9)? Since we don't wish to use JavaScript's imprecise Number for 0.9, we will instead first use fractions.js to precisely represent 0.9, then apply arctan:

    let ninth = fr.Fraction.make_fraction_from_string(`9/10`);  
    let b = cf.Constants.arctan(ninth);  

#### *Results!*

The `.to_decimal_string(precision-number-parameter)` operation does two things:

 1. Calculates the required precision.  
 2. Returns it as a string.

The calculation is encapsulated within the number object, and is retained there. Thus, once you've called it once for a given precision, future calls with the same/less precision should be rather quick.

OK, let's put this to practice: 

    let str1 = a.to_decimal_string(100);    
    let str2 = b.to_decimal_string(100);  
    console.log("pi: " + str1);  
    console.log("arctan(0.9): " + str2);  
    
### Example code 2: Arithmetic

    let cf = require("./continued_fractions.js");  
    let fr = require("./fractions.js");  

    let e = cf.Constants.e(); // the constant pi, i.e. 3.14159...  
    let two_thirds = cf.CF.make_cf_from_fraction(2n,3n); //wait, what? read on...  
    
    // Addition: e + (2/3)  
    let x1 = e.add(two_thirds);  
    console.log(`e + (2/3): ` + x1.to_decimal_string(100));  

    // Subtraction: e - (2/3)  
    let x2 = e.sub(two_thirds);  
    console.log(`e - (2/3): ` + x2.to_decimal_string(100));  

    // Division: e / (2/3), i.e. 3e/2  
    let x3 = e.div(two_thirds);  
    console.log(`e / (2/3): ` + x3.to_decimal_string(100));  

    // Multiplication: e * (2/3)  
    let x4 = e.mul(two_thirds);  
    console.log(`e * (2/3): ` + x4.to_decimal_string(100));  

    // Inverse: 1/e  
    let x5 = e.inverse();  
    console.log(`1/e: ` + x5.to_decimal_string(100));  
    
    // Square Root: sqrt(e)  
    let x6 = e.sqrt();  
    console.log(`sqrt(e): ` + x6.to_decimal_string(100));  
    
Now let's break that down:
 
#### *Two kinds of rationals? Yes, sorry...*

So... there are two kinds of rational number representations, as you might have noticed. The first, from *fractions.js*, should be used for only one purpose by you the user - as input for initializing constants from common functions, as we saw in **Example code 1**.  

The second is a rational number in the sense of a manipulable number with which we can perform arithmetic/print, as we see in this example for the variable two_thirds. For such purposes we initialize number-objects using cf.CF.make_cf_from_fraction(numerator, decominator);  

#### *The basic operations*

Other than the list above, the only arithmetics operation not mentioned is negate, i.e. multiplying by -1. It is currently unclear whether negative numbers always operate correctly, but feel free to test it if you so desire, for example -1*e: `let minus_e = e.negate; console.log(minus_e.to_decimal_string(100));`  

Given two number-objects, a and b, we can initialize a result number-object c as follows:  
Addition: `let c = a.add(b);`  
Subtraction: `let c = a.sub(b);`  
Division:  `let c = a.div(b);`  
Multiplication:  `let c = a.mul(b);`  
Multiplicative Inverse:  `let c = a.inverse();`  
Square Root:  `let c = a.sqrt();`  

### Remaining operations  

Although it makes no sense to me why you would do so, if you desire for the result to be a normal JavaScript Number, use `.to_float(decimal_precision)`. decimal_precision should be an integer.  

Given a number-objects you can deep-copy it by performing `.clone()`. Note that cloning means that computation done in one instance will not transfer to the other.  

To see a summary of the internal state of the calculation in a number-object, there are two-methods:  
`.to_text_string()`, recommended for deep analysis, which returns a full string description of the CF. Careful: Since we're dealing with very large BigInt's, it can get very big very quickly.  
`.to_text_string2()`, recommended for casual analysis, which returns a smaller string description of the CF. Note: unlike to_text_string, the description is imprecise.  

`.convergents(n, debug_if_this_is_slow_optional_argument)`, returns a series of fraction (from fr.Fractions) approximations ("convergents") by utilyzing the structure of a continued fraction.  

#### *efficient arithmetic*

The more "compound" a number-object is, that is to say - the more arithmetic operations were used in initializing it, the more difficult it will be to calculate new digits. Thus, while finding the first 15,000 digits of the constant e should take no more than 20 seconds, finding the even the first 100 digits of pi*pi + 3pi takes forever (long enough that I didn't bother to let it run its course). If we construct it as follows:  

    let pi = cf.Constants.pi();  
    let c = pi.mul(pi).add( pi.mul(cf.CF.make_cf_from_fraction(3,1)) );  
    console.log(c.to_decimal_string(100));  

There is a second reason why pi*pi + 3pi took so long: certain calculations take longer than others. In particular, any constant or trigonometric function based on an algebraic identity as a [Generalized Continued Fraction](https://en.wikipedia.org/wiki/Generalized_continued_fraction#Examples) is slower to converge, and that includes pi.  

So how about improving by a little bit (or a huge amount actually)?  

Given two number-objects `a` & `b`, and given 8 integers i1,...,i8, if we wish to find  
(i1*a*b + i2*a + i3*b + i4) /  (i5*a*b + i6*a + i7*b + i8),  

the most efficient way to do so is to initialize the result object "c" as follows:  

    let c = cf.CF.make_second_composite_cf(a,b,i1,i2,i3,i4,i5,i6,i7,i8)  

In fact, in general but not always, I implemented the four basic arithmetic operations (* + - /) using the same algorithm as `cf.CF.make_second_composite_cf`.    
Thus, for addition, the calculator calls cf.CF.make_second_composite_cf with i2=i3=i8=1, and the rest of the i's equal 0.  
For Division, the calculator calls cf.CF.make_second_composite_cf with i2=i7=1, and the rest of the i's equal 0.  
Etc.  

Let's return to pi*pi + 3pi, and let's aim higher this time - 1000 digits:  

    let pi = cf.Constants.pi();  
    let c = cf.CF.make_second_composite_cf(pi, pi, 1, 3, 0, 0, 0 , 0, 0, 1);  
    console.log(c.to_decimal_string(100));  
    
On my machine it took around 12 seconds. Much better, right?

Note: arithmetic operations between some number-object and a rational number object (one that is initialized using cf.CF.make_cf_from_fraction) are best performed using the previously introduced `.add, .sub, .div, .mul`, and **not** using the method make_second_composite_cf.

Another note on time-complexity: calculating the square root of anything other than a rational number-object is very intensive. Use `.sqrt` as rarely as possible.



***Amit Rubin + Mayer Goldberg, 2022***
