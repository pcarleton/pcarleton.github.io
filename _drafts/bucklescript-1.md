---
layout: post
title: "Getting started with BuckleScript"
---

I recently started playing around with [BuckleScript][1]. In this post I'll try to answer the following questions:

* What is BuckleScript?
* Why would I want to use it?
* How do I install it?
* How do I use it in practice?


# What is BuckleScript?

From the [website][1]:

>BuckleScript is a backend for OCaml which emits readable, modular and highly performant JavaScript.


Let me break that down.

#### OCaml
First, what is OCaml?  OCaml is a programming language that's been around for a long time.  Explaining OCaml's strengths and weaknesses deserves much more than paragraph in a blog post, so be sure to check out [ocaml.org][2] if you want a more thorough explanation.  Instead, I'll just relay my experience.

I had never programmed in OCaml before starting down this path.  I started with [Try OCaml][3] to get a feel for it.  Of the languages I am familiar with, it felt most similar to Haskell.  It supports functional style programming meaning that you write "pure" functions which have inputs and outputs, but no side effects.  OCaml feels a little less ideological than Haskell in that it supports "mutability" with just a keyword (no fussing around with Monads).  It also supports object-oriented programming in its own way.  I have found ["Real World OCaml"][4] to be an incredibly practical resource in seeing how OCaml works.

#### Backend
Now, what is a backend?  OCaml in its natural habitat supports a "byte" code backend and a "native" code backend.  This means it can take the same input OCaml code and emit a "byte" code representation or a "native" code representation. The "byte" code version is portable to multiple platforms.  It is interpreted by a virtual machine similar to Java bytecode.  The "native" code is a specific to a particular platform and is written in the instruction set for that platform.  It's execution is faster than the "byte" code version.  So this sentence is saying that BuckleScript is an additional backend for OCaml code which instead of "byte" code or "native" code, it emits JavaScript.


#### Readable, Modular
Next up is "readable, modular, and highly performant Javascript".  I'm not well versed in Javascript performance, so I can't comment on the "highly performant" aspect of it, so I'll focus on the "readable" and "modular" aspects.  "Readable" is a qualification that distinguishes BuckleScript from many other "compiled" Javascript languages.  I have dabbled with ClojureScript, Elm, and Typescript briefly in the past.  They behave differently than BuckleScript in that their compiled versions are very clearly compiled and often very difficult or impossible to read.  The output of BuckleScript on the other hand looks like Javascript that a human would conceivably write.  (This is true in the relatively simple examples I've seen, I haven't looked at anything very large or complex yet.  Also some things definitely look funky, like a "Record" actually being an "array" is probably not a decision you would make as a human.  Checkout some examples in the [manual][5] to see their outputs.)

For the "modular" piece, OCaml encourages the use of small "modules" (and you can even parameterize a module by another module using ["functors"][6] (Not to be confused with functors as seen in [Haskell][7])).  BuckleScript converts each OCaml module into its own JavaScript module which it then wires up using CommonJS or AMD as you prefer.  In other compiled JavaScript languages, the output is often one big ball of JS, so having things broken up like this is a neat change.

# Why would I want to use it?

Now that the "what" is covered, let's move on to the "why".  The short answer is if you really dislike JavaScript and you really like OCaml, then BuckleScript is for you.  The long answer comes in two parts.  The first is why use any compiled JavaScript framework, and the second is why use BuckleScript over any of the other options.

#### Why compiled JS?

First, I'll look at why you would want a compiled JavaScript language at all.  Adding another layer of indirection from the code you're writing to what gets executed is another place where things can go wrong.  Furthermore, going from one language to another is more complex than other layers you might have like minification or packing into a single file.  This means the failure modes will generally be tougher to negotiate.  So why bother?

For me, there are 2 main motivators for wanting a compiled language.  First, I like static analysis and type checking.  I like finding bugs before my code is ever run.  I like when the compiler can detect I did something that doesn't make sense.  Second, I find JavaScript's quirks annoying.  The ones I'm thinking of are "== vs. ===", "for (x in y)" behaving strangely, "undefined vs. null".  This one is not as strong as an argument since every language has its quirks, and once you get used to them, they're pretty easy to avoid.

#### Why BuckleScript?

If you're like me and are motivated to try a compiled JavaScript language the next question is which one to use.  The 2 things that set BuckleScript apart for me are the OCaml's support for functional programming the  interoperability and readability of the JavaScript output.

The OCaml type system and it's support for functional programming is something I really enjoy working with.  I find that when I write functional code, I spend more time up front thinking about the design of the system, and less time tracking down bugs. I like writing small functional blocks I feel confident about and then composing them together. It gives me comfort whereas writing imperative code to do the same thing leaves me wanting (I write mostly Python for my full time job).

The interoperability and readability go hand-in-hand.  I can reasonably guess what the JavaScript output of  BuckleScript is going to look like.  This means that when run-time bugs do happen, it is easier to determine what's causing them.  Additionally, it's easier to get the calls to external libraries to be exactly how you want them.

# How do I install it?

Congratulations if you've made it this far!  There a couple things to get you up and running with BuckleScript:

1. Install OCaml (and it's package manager)
2. Install the BuckleScript platform
3. Install editor tools

I basically followed the instructions from the [BuckleScript manual][8], but there were a couple additional steps.

#### Installing OCaml

To install ocaml, I used the following commands:

```
# Taken from https://opam.ocaml.org/doc/Install.html
sudo add-apt-repository ppa:avsm/ppa
sudo apt-get update
sudo apt-get install ocaml opam

opam init
opam update
opam switch 4.02.3+buckle-master

eval `opam config env`
```

The first stanza adds a repository that has the opam binary for Ubuntu.  The second stanza updates it, and switches to the OCaml compiler version used for BuckleScript.  "opam" is kind of like a combination between "pip" and "virtualenv" from the Python world.  That is, it finds and installs packages and also manages different environments with different versions of OCaml.  The last line sets up PATH variable.

#### Installing BuckleScript

To install bucklescript, navigate to the directory of your project. Then run:

```
npm install bs-platform
```

Note: This installation sets up some absolute paths.  If you end up moving your project directory somewhere else, you'll want to uninstall and re-install the bs-platform.

#### Installing Editor tools

The three tools I've been using for editing BuckleScript are Vim, Merlin, and Utop.

Vim is the editor, Merlin provides syntax highlighting (including errors) and auto complete, and Utop provides a repl.

The [Merlin wiki page][9] provides great installation instructions.

After installing Merlin, there is some extra configuration to do to get it to work with BuckleScript.  There is a section in the [BuckleScript FAQ][10].  Here is what my `.merlin` file looks like:

```
S src/
B lib/bs/src

B node_modules/bs-platform/lib/ocaml
S node_modules/bs-platform/lib/ocaml

FLG -ppx <absolute path to project>/node_modules/bs-platform/bin/bsppx.exe
```

Notably, I had to make the `FLG` line an absolute path.  When I was using the relative path, Merlin could not locate the `bsppx.exe` tool.  The `-ppx` argument is something to pass to the OCaml compiler in order to support syntax extensions.  This is what allows some of the variations on OCaml syntax that BuckleScript uses (like `##` and `[@bs]`).  To learn more about syntax extensions, check out [this helpful post][11].

# How do I use it in practice?

The [Getting Started][12] section of the manual is pretty good, but I'll add a little more info that I found helpful a long with a couple gotchas that may be obvious to more seasoned OCaml-ers, but tripped me up.

#### Structure of the project

I keep all my `.ml` files in a directory called `src`.  This is listed in my `bsconfig.json` file under `sources`.  Putting this in the `bsconfig.json` file lets the `bsb` command know where to look.  Then in my `package.json`, the `build` script is just `bsb`.  Running `npm run build` generates files under `lib`.  Under `lib/bs/src` are the build artifacts of `.cmi` and `.cmj` files (I'm not sure what exactly is in these files).  Under `lib/js/src` are all the JS source files.

#### Gotcha's

There were a couple mistakes I made that took some digging around to figure out what was going wrong.  I'll explain what they were in case other people run into them to.


#### When to use [@bs]
First, I was trying to call `Object.keys` on a JS object.  I tried this:

```
type 'a objHolder
let objVals : 'a objHolder -> 'a array [@bs] = [%bs.raw "Object.keys"]
```

but I kept getting an error at the call site like this:
```
let foo = objVals bar

This expression has type ([ `Arity_1 of 'a objHolder ], 'a0 array) Js.fn This is not a function; it cannot be applied.
```

I asked on the [discord channel][12], and here was the super helpful explanation I got from @chenglou:

>you need [@bs] at the function call site too
>
>the explanation is that if you annotate that function signature as [@bs], then instead of being a function BS will type it as 'actualArgs ThisIsJs conceptually. In which case it becomes not a function, which is the point (to avoid accidental currying overhead, etc.). So most of the time if you see that warning you know you forgot to aknowledge at the call site that it's indeed what you annotated it as: a non-curried js function call
>
>basically it's overloading the function calling syntax for other purposes
>
>also using let to define that will result in the error "type variable cannot be generalized" etc
>
>what you wanted is probably external objVals : 'a objHolder -> 'a array [@bs] = "Object.values" [@@bs.val].
>
>To do it with a proper ocaml let will be more troublesome
>
>buuuut also, iterating over all object keys/values is a metaprogramming technique from js that's not really supported
>
>which is also why ocaml (currently) has trouble printing arbitrary values for debugging (in bs you circumvent this by using Js.log which calls console.log under the hood "unsafely")
>
>Hope that made sense

So the actual way to call `Object.keys` is:
```
 external objVals : 'a objHolder -> 'a array [@bs] = "Object.keys" [@@bs.val]
```

(I was trying to call `Object.values` before I realized that it is not broadly supported.)

#### Visibility of Record Fields

The next bit I stumbled on was in setting the fields of a record.  For example, if I have:

```
type Person = {name : string; age : int }
```

in one module and in another module I try to modify a "Person" record in another module, the compiler will complain that doesn't know about the "name" field.  To remedy this, you can do:

```
module Person = struct
  type t = {name: string; age: int}
end
```

then before modifying the record, you can call `open <base module>.Person`.


#### The `Core` module

When following along with "Real World OCaml", I tried using the `Core` module thinking it was part of the standard OCaml library and therefore available to me in Bucklescript.  It turns out the `Core` module is a third party library.  This was relatively minor but caused me some confusion. (and [apparently][13] I'm not the only one!)


#### Match statements and pipes

I was scratching my head for a while over a Match expression that kept telling me I hadn't covered all the cases when I clearly had.  I moved things around a bunch before realizing I had forgot to add the pipe character (`|`) before each of my cases.  This one was a bit of a facepalm moment, but maybe it will save someone a headache.


# Conclusion

Hopefully this post gave you a good run down of BuckleScript.  In a future post, I'll dive into what I decided to build for my first project (hint: it's related to the game [Screeps][15]).  Feel free to [contact me][14] or leave a comment if I totally botched something or if you have questions.


[1]:https://bloomberg.github.io/BuckleScript/
[2]:http://www.ocaml.org/learn/description.html
[3]:https://try.ocamlpro.com/
[4]:https://realworldocaml.org
[5]:https://bloomberg.github.io/BuckleScript/Manual.html
[6]:https://realworldocaml.org/v1/en/html/functors.html
[7]:http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html
[8]:https://bloomberg.github.io/bucklescript/Manual.html#__strong_recommended_strong_installation_with_opam
[9]:https://github.com/ocaml/merlin/wiki/vim-from-scratch
[10]:https://bloomberg.github.io/bucklescript/Manual.html#_faq
[11]:https://whitequark.org/blog/2014/04/16/a-guide-to-extension-points-in-ocaml/
[12]:https://discordapp.com/invite/reasonml
[13]:https://github.com/bloomberg/bucklescript/issues/901
[14]:https://twitter.com/paulcarletonjr
[15]:https://screeps.com
