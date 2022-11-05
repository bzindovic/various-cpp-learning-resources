# Hardware, optimization, and OS internals <!-- omit in toc -->

## Table of contents <!-- omit in toc -->

- [Hardware](#hardware)
	- [CPU](#cpu)
		- [CPU word size](#cpu-word-size)
		- [Endianness and NUXI problem](#endianness-and-nuxi-problem)
		- [x87](#x87)
	- [Memory](#memory)
		- [Memory addressing](#memory-addressing)
	- [Device drivers](#device-drivers)
- [Optimizations](#optimizations)
	- [Branch prediction](#branch-prediction)
		- [`[[likely]]` / `[[unlikely]]` attributes](#likely--unlikely-attributes)
		- [`likely` / `unlikely` Linux kernel macros](#likely--unlikely-linux-kernel-macros)
	- [Memory](#memory-1)
		- [Memory copying](#memory-copying)
		- [Memory allocation](#memory-allocation)
		- [Memory relocation](#memory-relocation)
		- [Memory access](#memory-access)
		- [Nested `std::vector`s](#nested-stdvectors)
	- [Floating-point arithmetic](#floating-point-arithmetic)
	- [Integral arithmetic](#integral-arithmetic)
		- [Integral multiplication](#integral-multiplication)
		- [Integeral division](#integeral-division)
	- [Empty base class optimization](#empty-base-class-optimization)
	- [Return value optimization and copy elision](#return-value-optimization-and-copy-elision)
	- [Tail call optimisation](#tail-call-optimisation)
	- [Devirtualization](#devirtualization)
	- [Undefined behavior](#undefined-behavior)
		- [Infinite loops](#infinite-loops)
		- [Strict aliasing rule](#strict-aliasing-rule)
- [Benchmarking](#benchmarking)
- [Profiling](#profiling)

---

## Hardware

:link:

- B.Wagstaff. [*A journey through the CPU pipeline*](https://www.gamedev.net/articles/programming/general-and-gameplay-programming/a-journey-through-the-cpu-pipeline-r3115/) (2013)
- U.Drepper. [*What every programmer should know about memory*](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf) (2007)
- [*Simple benchmark for memory throughput and latency*](https://github.com/ssvb/tinymembench)
- L.Maranget, S.Sarkar, P.Sewell. [*A tutorial introduction to the ARM and POWER relaxed memory models*](https://www.cl.cam.ac.uk/~pes20/ppc-supplemental/test7.pdf) (2012)

:movie_camera:

- C.Terman. *Virtual memory.* [Part I](https://www.youtube.com/watch?v=3akTtCu_F_k), [Part II](https://www.youtube.com/watch?v=DelO8tZFMrc) – MIT 6.004: Computation structures (2013)

<!-- https://web.archive.org/web/20080107035604/http://www.cellperformance.com/mike_acton/2006/05/demystifying_the_restrict_keyw.html -->
<!-- - O.Mutlu. [Lec. 20: *Virtual memory*](https://www.youtube.com/watch?v=2RhGMpY18zw) – - Comp. Arch. 2015 -->

### CPU

#### CPU word size

:link:

- S.Ignatchenko. [*Size matters*](https://accu.org/journals/overload/22/120/ignatchenko_1895/) – [Overload **120**](https://accu.org/journals/overload/overload120) (2014)
- E.Musayev. [*A brief history of the road to 64 bits*](https://web.archive.org/web/20191222085303/http://www.eldar.com/node/262) (in Russian, 2009)

:page_facing_up:

- J.Mashey. [*The long road to 64 bits*](https://cacm.acm.org/magazines/2009/1/15667-the-long-road-to-64-bits/fulltext) – [Communications of the ACM **52**, 45-53](https://doi.org/10.1145/1435417.1435431) (2009)

#### Endianness and NUXI problem

:link:

- [*Endianness*](https://en.wikipedia.org/wiki/Endianness) – Wikipedia

:book:

- Essay 1: *You must be joking* – P.J.Plauger. [*Programming on purpose III: Essays on software technology*](https://www.pearson.com/us/higher-education/program/Plauger-Programming-on-Purpose-III-Essays-on-Software-Technology/PGM133229.html) (1994)

#### x87

See also [*Numeric data structures and algorithms*](../data_structures_and_algorithms/numeric.md).

:link:

- [*Pentium `FDIV` bug*](https://en.wikipedia.org/wiki/Pentium_FDIV_bug#cite_note-halfhill-199503-3) – Wikipedia
- B.Dawson. [*Intel underestimates error bounds by 1.3 quintillion](https://randomascii.wordpress.com/2014/10/09/intel-underestimates-error-bounds-by-1-3-quintillion/) (2014)
- S.Duplichan. [*Intel overstates FPU accuracy*](http://notabs.org/fpuaccuracy/index.htm) (2013)
- T.R.Halfhill. [*The truth behind the Pentium bug*](https://web.archive.org/web/20060209005434/http://www.byte.com/art/9503/sec13/art1.htm) – BYTE.com (1995)

:grey_question:

- [*Extended (80-bit) double floating point in x87, not SSE2 – we don’t miss it?*](https://stackoverflow.com/q/3206101) – Stack Overflow
- [*Did any compiler fully use Intel x87 80-bit floating point?*](https://retrocomputing.stackexchange.com/q/9751) – Retrocomputing

:anchor:

- [*Programming with the x87 floating-point unit*](http://www.infophysics.net/x87.pdf) – Intel

### Memory

:link:

- E.Martin. [*Some things I’ve learned about memory*](http://neugierig.org/software/blog/2011/05/memory.html) (2011)

:movie_camera:

- U.Drepper. [*C++ and memory: Between correctness and performance*](https://www.youtube.com/watch?v=LXfSXzxDY_M) – code::dive (2018)

#### Memory addressing

:link:

- D.A.Rusling. [Ch. 3: *Memory management*](http://www.tldp.org/LDP/tlk/mm/memory.html) – [The Linux kernel](http://www.tldp.org/LDP/tlk/tlk-title.html)
- C.Santili. [*x86 paging tutorial*](https://cirosantilli.com/x86-paging)
- [*How does x86 paging work?*](https://stackoverflow.com/q/18431261) – Stack Overflow
- [*What are near, far and huge pointers?*](https://stackoverflow.com/q/3575592) – Stack Overflow

:book:

- Essay 1: *You must be joking* – P.J.Plauger. [*Programming on purpose III: Essays on software technology*](https://www.pearson.com/us/higher-education/program/Plauger-Programming-on-Purpose-III-Essays-on-Software-Technology/PGM133229.html) (1994)

<!-- https://web.archive.org/web/20080107035604/http://www.cellperformance.com/mike_acton/2006/05/demystifying_the_restrict_keyw.html -->

<!-- https://www.airs.com/blog/archives/120
https://www.agner.org/optimize/optimizing_cpp.pdf
https://www.agner.org/optimize/
http://www.reedbeta.com/blog/data-oriented-hash-table/
 -->

### Device drivers

:movie_camera:

- D.Saks. [*Memory-mapped devices as objects*](https://www.youtube.com/watch?v=uwzuAGtAEFk) – CppCon (2020)

---

## Optimizations

See also [*Optimizations* – Compilers](../data_structures_and_algorithms/parsing.md#optimizations).

:link:

- [*Optimizing C++*](https://en.wikibooks.org/wiki/Optimizing_C%2B%2B) – WikiBooks
- A.O’Dwyer. [*`[[trivial_abi]]` 101*](https://quuxplusone.github.io/blog/2018/05/02/trivial-abi-101/)

:grey_question:

- [*Can `const`-correctness improve performance?*](https://stackoverflow.com/q/3435026) – Stack Overflow
- [*Why does this loop produce “warning: iteration 3u invokes undefined behavior” and output more than 4 lines?*](https://stackoverflow.com/q/24296571) – Stack Overflow
- [*What is tail call optimization?*](https://stackoverflow.com/q/310974) – Stack Overflow
- [*Is premature optimization really the root of all evil?*](https://softwareengineering.stackexchange.com/q/80084) – Software Engineering

:movie_camera:

- A.Alexandrescu. [*Speed is found in the minds of people*](https://www.youtube.com/watch?v=FJJTYQYB1JQ) – CppCon (2019)
- H.Matthews. [*Optimising a small real-world C++ application*](https://www.youtube.com/watch?v=fDlE93hs_-U) – ACCU (2019)
- H.Matthews. [*Optimising a small real-world C++ application*](https://www.youtube.com/watch?v=IGFBCvroXJ8) – NDC (2018)
- H.Matthews. [*C++ performance and optimisation*](https://www.youtube.com/watch?v=eICYHA-eyXM) – NDC (2017)
- F.Pikus. [*Design for performance*](https://www.youtube.com/watch?v=m25p3EtBua4) – CppCon (2018)
- C.Bay. [*The CPU cache: Instruction re-ordering made obvious*](https://www.youtube.com/watch?v=tNkVUIv2gEE) – C++Now (2016)
- C.Carruth. [*Understanding compiler optimization*](https://www.youtube.com/watch?v=haQ2cijhvhE) – code::dive (2016)
- C.Cook. [*The speed game: Automated trading systems in C++*](https://www.youtube.com/watch?v=ulOLGX3HNCI) – Meeting C++ (2016)
- T.Doumler. [*Want fast C++? Know your hardware!*](https://www.youtube.com/watch?v=BP6NxVxDQIs) – CppCon (2016)
- A.Alexandrescu. [*Optimization tips*](https://www.youtube.com/watch?v=Qq_WaiwzOtI) – CppCon (2014)
- M.Godbolt. [*x86 internals for fun & profit*](https://www.youtube.com/watch?v=hgcNM-6wr34) – GOTO (2014)
- S.Meyers. [*CPU caches and why you care*](https://www.youtube.com/watch?v=WDIkqP4JbkE) – code::dive (2014)

:book:

- Col. 9: *Code tuning* – J.Bentley. [*Programming pearls*](https://www.oreilly.com/library/view/programming-pearls-second/9780134498058/) (1999)

:anchor:

- [*Intel 64 and IA-32 architectures optimization reference manual*](https://www-ssl.intel.com/content/www/us/en/architecture-and-technology/64-ia-32-architectures-optimization-manual.html)

### Branch prediction

:link:

- [*Branch predictor*](https://en.wikipedia.org/wiki/Branch_predictor) – Wikipedia

#### `[[likely]]` / `[[unlikely]]` attributes

> These attributes allow the compiler to optimize for the case where paths of execution are more or less likely than any alternative path of execution.

:grey_question:

- [*How to use C++20’s `likely`/`unlikely` attributes in `if-else` statement*](https://stackoverflow.com/q/51797959) – Stack Overflow

<!-- :movie_camera: -->

:anchor:

- [*C++ attribute: `likely`, `unlikely`*](https://en.cppreference.com/w/cpp/language/attributes/likely) – C++ reference

#### `likely` / `unlikely` Linux kernel macros

:grey_question:

- [*How do the `likely`/`unlikely` macros in the Linux kernel work and what is their benefit?*](https://stackoverflow.com/q/109710) – Stack Overflow

### Memory

:link:

- J.M&uuml;ller. [*`malloc()` and `free()` are a bad API*](https://www.foonathan.net/2022/08/malloc-interface/) (2022)

#### Memory copying

:link:

- [*Time to revisit `REP MOVS`*](https://software.intel.com/en-us/forums/intel-fortran-compiler/topic/275765) – Intel Developer Zone (2006)

:grey_question:

- [*Enhanced `REP MOVSB` for `memcpy`*](https://stackoverflow.com/q/43343231) – Stack Overflow
- [*What setup does `REP` do?*](https://stackoverflow.com/q/33902068) – Stack Overflow
- [*Why are complicated `memcpy`/`memset` superior?*](https://stackoverflow.com/q/8858778) – Stack Overflow

#### Memory allocation

See also [*Allocators* – The standard library, Boost, and proposals](std_library.md#allocators).

:link:

- N.Fitzgerald. [*Always bump downwards*](https://fitzgeraldnick.com/2019/11/01/always-bump-downwards.html) (2019)

:grey_question:

- [*Is the compiler allowed to optimize out heap memory allocations?*](https://stackoverflow.com/q/31873616) – Stack Overflow

:movie_camera:

- J.Lakos. *Local (“arena”) memory allocators.* [Part I](https://www.youtube.com/watch?v=nZNd5FjSquk), [Part II](https://www.youtube.com/watch?v=CFzuFNSpycI) – CppCon (2017)
- J.Lakos. [*Local (“arena”) memory allocators*](https://www.youtube.com/watch?v=d1DpVR0tw0U) – ACCU (2017)
- A.Alexandrescu. [*`std::allocator` is to allocation what `std::vector` is to vexation*](https://www.youtube.com/watch?v=LIb3L4vKZ7U) – CppCon (2015)

#### Memory relocation

:link:

- A.O’Dwyer. [*Announcing “trivially relocatable”*](https://quuxplusone.github.io/blog/2018/07/18/announcing-trivially-relocatable/) (2018)

:movie_camera:

- A.O’Dwyer. [*Trivially relocatable*](https://www.youtube.com/watch?v=SGdfPextuAU) – C++Now (2019)

:anchor:

- A.O’Dwyer. [*Object relocation in terms of move plus destroy*](https://wg21.link/p1144) – WG21/P1144

#### Memory access

:grey_question:

- [*Why is transposing a matrix of `512x512` much slower than transposing a matrix of `513x513`?*](https://stackoverflow.com/q/11413855) – Stack Overflow
- [*Why are elementwise additions much faster in separate loops than in a combined loop?*](https://stackoverflow.com/q/8547778) – Stack Overflow
- [*Why don’t C++ compilers optimize this conditional boolean assignment as an unconditional assignment?*](https://stackoverflow.com/q/40303182) – Stack Overflow

#### Nested `std::vector`s

:grey_question:

- [*Is it a good idea to use `vector<vector<double>>` to form a matrix class for high performance scientific computing code?*](https://scicomp.stackexchange.com/questions/3159/is-it-a-good-idea-to-use-vectorvectordouble-to-form-a-matrix-class-for-high/3162) – Computational Science
- [*Performance impact of nested vectors vs. contiguous arrays*](https://stackoverflow.com/q/45747848) – Stack Overflow
- [*Using nested vectors vs a flatten vector wrapper, strange behaviour*](https://stackoverflow.com/q/33093860) – Stack Overflow

### Floating-point arithmetic

:link:

- K.Walfridsson. [*Optimizations enabled by `-ffast-math`*](https://kristerw.github.io/2021/10/19/fast-math/) (2021)

:grey_question:

- [*What does gcc’s `ffast-math` actually do?*](https://stackoverflow.com/q/7420665) – Stack Overflow

:anchor:

- [Bug 323: *Optimized code gives strange floating point results*](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=323) – GCC Bugzilla

### Integral arithmetic

:grey_question:

- [*What is the performance impact of using `int64_t` instead of `int32_t` on 32-bit systems?*](https://stackoverflow.com/q/16841382) – Stack Overflow

#### Integral multiplication

:grey_question:

- [*Why is `imul` used for multiplying unsigned numbers?*](https://stackoverflow.com/q/42587607) – Stack Overflow

#### Integeral division

:link:

- D.W.Jones. [*Reciprocal multiplication, a tutorial*](https://homepage.divms.uiowa.edu/~jones/bcd/divide.html) (1999)
- T.Granlund, P.L.Montgomery. [*Division by invariant integers using multiplication*](https://gmplib.org/~tege/divcnst-pldi94.pdf) (1994)

:grey_question:

- [*Why does GCC use multiplication by a strange number in implementing integer division?*](https://stackoverflow.com/q/41183935) – Stack Overflow
- [*Why does the compiler generate a right-shift by 31 bits when dividing by 2?*](https://stackoverflow.com/q/40638335) – Stack Overflow

### Empty base class optimization

> Empty base class optimization allows the size of an empty base subobject to be zero. Empty base optimization is required for standard layout types.
> ```cpp
> struct Empty {};
> static_assert(sizeof(Empty) >= 1);
>
> struct Derived : Empty {
>     T i;
> };
> static_assert(sizeof(Derived) == sizeof(T));
> ```

:link:

- N.C.Myers. [*The “empty member” C++ optimization*](https://www.drdobbs.com/cpp/the-empty-member-c-optimization/184410250) ([mirror](http://www.cantrip.org/emptyopt.html)) – Dr.Dobb’s Journal (1997)
- [*Empty base optimization*](https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Empty_Base_Optimization) – WikiBooks
- [*`boost::compressed_pair`*](https://www.boost.org/doc/libs/release/libs/utility/compressed_pair.htm)
- [*`boost::empty_value`*](https://www.boost.org/doc/libs/release/libs/core/doc/html/core/empty_value.html)

:book:

- Sec. 21.1: *The empty base class optimization*, Sec. 25.5.1: *Tuples and the EBCO* – D.Vandevoorde, N.M.Josuttis, D.Gregor. [*C++ templates: The complete guide*](http://www.tmplbook.com/) – [Addison-Wesley](https://www.informit.com/store/c-plus-plus-templates-the-complete-guide-9780321714121) (2017)

:anchor:

- [*Empty base optimization*](https://en.cppreference.com/w/cpp/language/ebo) – C++ reference

### Return value optimization and copy elision

:link:

- A.Fertig. [*Why you should use `std::move` only rarely*](https://andreasfertig.blog/2022/02/why-you-should-use-stdmove-only-rarely/) (2022)

:camera:

- A.O’Dwyer. [*The complete guide to `return x;`*](https://www.youtube.com/watch?v=OGKAJD7bmr8) – C++Now (2021)
- J.Kalb. [*Copy elision*](https://www.youtube.com/watch?v=fSB57PiXpRw) – C++Now (2018)

:anchor:

- [*Copy elision*](https://en.cppreference.com/w/cpp/language/copy_elision) – C++ reference

### Tail call optimisation

:link:

- A.O’Dwyer. [*It’s not always obvious when tail-call optimization is allowed*](https://quuxplusone.github.io/blog/2021/01/09/tail-call-optimization/) (2021)
- A.Balaam. [*Tail call optimisation in C++*](https://accu.org/journals/overload/20/109/balaam_1914/) – [Overload **109**](https://accu.org/journals/overload/overload109) (2012)

### Devirtualization

- A.O’Dwyer. [*When can the C++ compiler devirtualize a call?*](https://quuxplusone.github.io/blog/2021/02/15/devirtualization/) (2021)

### Undefined behavior

:link:

- K.Walfridsson. [*How undefined signed overflow enables optimizations in GCC*](https://kristerw.blogspot.com/2016/02/how-undefined-signed-overflow-enables.html) (2016)
- K.Walfridsson. [*Dangling pointers and undefined behavior*](https://kristerw.blogspot.com/2016/04/dangling-pointers-and-undefined-behavior.html) (2016)
- K.Walfridsson. [*Pointer comparison — an invalid optimization in GCC*](https://kristerw.blogspot.com/2016/12/pointer-comparison-invalid-optimization.html) (2016)
- J.Regehr. [*Finding undefined behavior bugs by finding dead code*](https://blog.regehr.org/archives/970) (2013)
- C.Lattner. *What every C programmer should know about undefined behavior.* [Part I](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html), [Part II](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know_14.html), [Part III](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know_21.html) – LLVM project (2013)
- O.Maudel. [*Demons may fly out of your nose*](https://accu.org/journals/overload/21/115/maudel_1857/) – [Overload **115**](https://accu.org/journals/overload/overload115) (2013)
- M.Shroyer. [*Both true and false: A Zen moment with C*](https://markshroyer.com/2012/06/c-both-true-and-false/) (2012)

<!-- http://blog.regehr.org/archives/213 -->

:movie_camera:

- A.Sermersheim, B.Geller. [*Back to basics: Undefined behavior*](https://www.youtube.com/watch?v=NpL9YnxnOqM) – CppCon (2021)
- J.Regehr. [*Undefined behavior and compiler optimizations*](https://www.youtube.com/watch?v=AeEwxtEOgH0) – C++Now (2018)
- B.Geller, A.Sermersheim. [*Undefined behavior is not an error*](https://www.youtube.com/watch?v=XEXpwis_deQ) – CppCon (2018)
- P.Padlewski. [*Undefined behaviour is awesome!*](https://www.youtube.com/watch?v=ehyHyAIa5so) – CppCon (2017)
- M.Spencer. [*My little optimizer: Undefined behavior is magic*](https://www.youtube.com/watch?v=g7entxbQOCc) – CppCon (2016)

#### Infinite loops

See also [*Infinite loop* – Patterns, idioms, and design principles](patterns_and_idioms.md#infinite-loop).

:grey_question:

- [*Optimizing away a `while(1);` in C++0x*](https://stackoverflow.com/q/3592557) – Stack Overflow

:anchor:

- H.-J. Boehm. [*Why undefined behavior for infinite loops?*](https://wg14.link/n1528) – WG14/N1528

#### Strict aliasing rule

See [*Type-punning* – Core language](core_language.md#type-punning).

---

## Benchmarking

:link:

- D.Ferenc. [*Contractual loopholes*](https://accu.org/journals/overload/25/138/ferenc_2363/) – [Overload **138**](https://accu.org/journals/overload/overload138) (2017)

---

## Profiling

:movie_camera:

- M.Ropert. [*The basics of profiling*](https://www.youtube.com/watch?v=dToaepIXW4s) – CppCon (2021)
