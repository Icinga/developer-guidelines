
### Preparations <a id="development-develop-prepare"></a>

#### Choose your Editor <a id="development-develop-choose-editor"></a>

Icinga 2 can be developed with your favorite editor. Icinga developers prefer
these tools:

- vim
- CLion (macOS, Linux)
- MS Visual Studio (Windows)
- Atom

Editors differ on the functionality. The more helpers you get for C++ development,
the faster your development workflow will be.

#### Get to know the architecture <a id="development-develop-get-to-know-the-architecture"></a>

Icinga 2 can run standalone or in distributed environments. It contains a whole lot
more than a simple check execution engine.

Read more about it in the [Technical Concepts](19-technical-concepts.md#technical-concepts) chapter.

#### Get to know the code <a id="development-develop-get-to-know-the-code"></a>

First off, you really need to know C++ and portions of C++11 and the boost libraries.
Best is to start with a book or online tutorial to get into the basics.
Icinga developers gained their knowledge through studies, training and self-teaching
code by trying it out and asking senior developers for guidance.

Here's a few books we can recommend:

* [Accelerated C++: Practical Programming by Example](https://www.amazon.com/Accelerated-C-Practical-Programming-Example/dp/020170353X) (Andrew Koenig, Barbara E. Moo)
* [Effective C++](https://www.amazon.com/Effective-Specific-Improve-Programs-Designs/dp/0321334876) (Scott Meyers)
* [Boost C++ Application Development Cookbook - Second Edition: Recipes to simplify your application development](https://www.amazon.com/dp/1787282244/ref=cm_sw_em_r_mt_dp_U_dN1OCbERS00EQ) (Antony Polukhin)
* [Der C++ Programmierer](https://www.amazon.de/Programmierer-lernen-Professionell-anwenden-L%C3%B6sungen/dp/3446416447), German (Ulrich Breymann)
* [C++11 programmieren](https://www.amazon.de/gp/product/3836217325/), German (Torsten T. Will)

In addition, it is a good bet to also know SQL when diving into backend development.

* [SQL Performance Explained](https://www.amazon.de/gp/product/3950307826/) (Markus Winand)

Last but not least, if you are developing on Windows, get to know the internals about services and the Win32 API.
