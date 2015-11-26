Lisp to Lua Compiler
====================

[![Join the chat at https://gitter.im/meric/l2l](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/meric/l2l?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

A Lisp to Lua compiler, compatible with LuaJIT or Lua5.1+. Lua 5.2+ or 
LuaJIT (built with -DLUAJIT_ENABLE_LUA52COMPAT) recommended for higher
performance.


Features
-----------
* Reader Macros
* Macros
* Lua functions
* Compiler modification on-the-fly during compile-time..
* Compiler partially implemented by itself. See https://github.com/meric/l2l/blob/master/compiler.lua#L596


Contribute
----------
Play around. Make issues. Submit pull requests. :)


How To
------

* `./bin/l2l` to launch REPL.

        ;; Welcome to Lua-To-Lisp REPL!
        ;; Type '(print "hello world!") to start.
        >> (print "Hello world!")
        Hello world!
        =   nil
        >> 

* `./bin/l2l --enable read_execute sample01.lsp` to compile `sample01.lsp` and output Lua to stdout.
* `./bin/l2l --enable read_execute sample01.lsp | lua` to compile and run `sample01.lsp` immediately.
* `./bin/l2l sample02.lsp sample03.lsp` to compile two lisp files where one 
    requires the other.
* `make -C sample04` to run the makefile in the sample04 directory. It
   demonstrates how to use l2l from another directory.

Use in Lua
----------
        
        lua
        Lua 5.2.4  Copyright (C) 1994-2015 Lua.org, PUC-Rio
        > require("l2l.eval").loadstring("(print 1) (print 2)")
        > require("l2l.eval").loadstring("(print 1) (print 2)")()
        1
        2
        > 


Differences from other lisps
----------------------------

While l2l is not a Scheme or Common Lisp implementation, it shares
many features of these languages.

It is a [Lisp-1](https://hornbeck.wordpress.com/2009/07/05/lisp-1-vs-lisp-2/)
like Scheme, which means that functions are treated like other values
rather than being stored in a separate namespace. However, the macro
system uses a much simpler `defmacro` like CL and Clojure instead of
Scheme's hygenic macros.

The `let` macro for binding locals works more like Clojure--it does
not require each name/value pair to be wrapped in its own parens:

```lisp
(let (x 12
      y 30)
  (+ x (* y 22)))
```

The syntax for [varargs](https://en.wikipedia.org/wiki/Variadic_function)
is taken from Lua rather than any existing lisp dialect; it uses three dots:

```lisp
(defun myfun (...) (cdr (pack ...)))
```

The `-` and `/` operators do not have a unary mode. `(- 4)` and `(/ 4)` both
returns 4. Implementing unary mode would prevent implementing these two
operators directly in the form of `(a - b - c - d...)` and `(a / b / c / d)`.
There are complications that can arise because of Lua's vararg mechanics - 
Should `(- (somefunction x))` be a unary call or a non-unary call? `somefunction`
can return 1 or more values, and it is impossible to know which, in the 
compiling stage, before the particular call is evaluated.

Internals
---------

* Change prompt string by setting the `_P` global variable.

        >> (set _P ">->o ")
        =   >> 
        >->o (print "Hello World")
        Hello World
        =   nil
        >->o 

* The read macro table is `_R`. `_R.META` stores locations of all read symbols.

        >> (set _R.META {}) ;; _R.META is too big.
        =   table: 0x7fcf90c54c90
        >> (show _R)
        =   {"}" function: 0x7fcf90c1a930 "META" {(show _R) {1 6 2 9 0 6}} ";" function: 0x7fcf90c1a8c0 "position" function: 0x7fcf90c1b1a0 ")" function: 0x7fcf90c1a8e0 "(" function: 0x7fcf90c1ac10 "'" function: 0x7fcf90c1ad30 "," function: 0x7fcf90c1adb0 "[" function: 0x7fcf90c1aab0 "#" function: 0x7fcf90c1adf0 "\"" function: 0x7fcf90c1a9d0 "]" function: 0x7fcf90c1a980 "`" function: 0x7fcf90c1ad70 "{" function: 0x7fcf90c1ab80}
        >> 

* The dispatch read macro table is `_D`.

        >> (show _D)
        =   {"." function: 0x7fcf90c27360 " " function: 0x7fcf90c1a890 "'" function: 0x7fcf90c1acf0}

* The compiler table is `_C`.

        >> (show _C)
        =   {"_60_61" function: 0x7fac1bc1e4d0 "_105_102" function: 0x7fac1bc948f0 "_" function: 0x7fac1bc26280 "defcompiler" function: 0x7fac1bc266a0 "defun" function: 0x7fac1bc26820 "_111_114" function: 0x7fac1bc261c0 "quasiquote" function: 0x7fac1bc26460 "_119_104_105_108_101" function: 0x7fac1bc3b300 "cadr" function: 0x7fac1bc26780 "_35" function: 0x7fac1bc26370 "_100_111" function: 0x7fac1bc64040 "car" function: 0x7fac1bc26720 "_58" function: 0x7fac1bc26330 "_62" function: 0x7fac1bc1e570 "cond" function: 0x7fac1bc264b0 "_47" function: 0x7fac1bc262c0 "chunk" function: 0x7fac1bc4a340 "_46_46" function: 0x7fac1bc0be00 "_43" function: 0x7fac1bc26240 "_110_111_116" function: 0x7fac1bc26190 "_46" function: 0x7fac1bc26300 "_62_61" function: 0x7fac1bc26090 "quote" function: 0x7fac1bc26420 "_98_114_101_97_107" function: 0x7fac1bc6bc30 "set" function: 0x7fac1bc263a0 "table_quote" function: 0x7fac1bc263e0 "_61_61" function: 0x7fac1bc1e3d0 "defmacro" function: 0x7fac1bc717b0 "let" function: 0x7fac1bc267b0 "_42" function: 0x7fac1bc26200 "lambda" function: 0x7fac1bc26650 "_60" function: 0x7fac1bc1e450 "_97_110_100" function: 0x7fac1bc26130 "cdr" function: 0x7fac1bc26750}
        >> 

* The format of a compiler is a function with at least two arguments.
    For example: 

        local function compile_subtract(block, stream, ...)
          return "("..map(bind(compile, block, stream), {...}):concat(" - ")..")"
        end

    This implements compiling `(- 1 2 3)` to `(1 - 2 - 3)`.

    A compiler function inserts any non-expression Lua statements into `block`,
    and returns a single Lua expression which should either be the value
    or reference to the value that will be returned in its parent lisp block
    (which is likely to be a lisp function).

    The arguments of a function are raw lisp values, uncompiled and 
    unprocessed. They must be compiled before being inserted into generated 
    Lua code.

* Use the `defcompiler` helper to define compilers in lisp. 
    For example:

        (defcompiler -- (block stream str)
            (table.insert block (.. "\n--" (tostring str))))

    This implements comments that will be printed directly into the Lua output.

    `defcompiler` will put your compiler into `_C` table as well as activate
    the compiler immediately for use. Right after the above compiler 
    declaration you can have:

        `(-- "This is a comment")`

    and the code will be output directly as "-- This is a comment" into the
    Lua source code.
    
    The following information about `defcompiler` is relevant for compilers
    taking variable arguments, i.e. `...`, like the add operator.
    `(+ 1 2 3...)`:

* `defmacro` is currently implemented like so:


        (defcompiler defmacro (block stream name parameters ...)
          (let 
            (params (list.push (list.push parameters 'stream) 'block)
             code `(defcompiler ,name ,params
                     (let (fn (eval `(lambda ,parameters ,...)))
                       (compile block stream (fn ,(list.unpack parameters))))))
            (eval code)
            (compile block stream code)))


TODO
----

* Make sure `_R.META` is recording locations accurately enough during the compiler 
stage.
* ~~Implement a method to automate unwrapping of `...` arguments to operators.~~
* `compiler.lua` self-bootstrapping generates ugly code and poses problem when sandboxing. 
* Replace the io interface `reader.lua` uses with one that has nothing to do with files.

License
=======

Copyright © 2012-2015, Eric Man and contributors
Released under the 2-clause BSD license, see LICENSE

