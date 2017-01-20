---
layout: post
title: "Buckle Typings - Creating modules useable in BuckleScript from TypeScript typings"
---

In my [last post][1], I put together a simple (and incomplete) wrapper around PixiJS in order to use it in BuckleScript.  This is a common task when using untyped code in a typed system; you have to specify the types. TypeScript has to do this as well.  These type definitions take work to get right, so it would be really nice if you could re-use the work the TypeScript community has put in to creating definitions for many popular libraries.

I found a mention of this exact idea on [an issue][2] for the `reason-js` project.  I asked around in the Reason discord channels and there didn't seem to be an active effort in this direction.  Someone pointed me to [flowgen][3] which is an attempt to convert TypeScript typings into something that can be used by the Flow type checker.

Interestingly, that project works by calling directly into the TypeScript parser.  The parser gives you an AST, so traversing the nodes and executing on particular ones is relatively straightforward.  This is an idea I hadn't considered since I was planning on parsing the definitions in OCaml.  This makes a lot of sense though because it lets TypeScript, which knows this stuff best, do the heavy lifting.


The `flowgen` project builds up these nodes and then has a series of "printers" which know how to print the "flow-style" type from the information collected in the node.  This seems like a reasonable approach that might also work with BuckleScript.

# The First Target

The first thing I'll need in order to get started is some golden output.  I'll start with a very simple typings definition, pick what I would like the OCaml module to look like, and then do the minimum amount to get that output.  I'll start with a package with a single integer variable exported.  I looked at the [TypeScript documentation][4] for how to do this, and it looks like this:

```
declare var foo: number;
```

The desired BuckleScript output is :

```
external foo: int = "foo" [@@bs.val];
```

# Parsing the File

Next I'll want to create TypeScript AST using the TypeScript compiler.  I adapted the following from the `flowgen` project, adjusting to use ES5 (see note below about this Enum):

```
var typescript = require('typescript');

var sourceFile = typescript.createSourceFile("dummy_path",
  "declare var foo: number;",
  typescript.ScriptTarget.ES5,
  false
);

console.log(sourceFile);
```

When I run `node compiler.js`, I get a `SourceFileObject` printed to the screen.

Next, I want to parse the file using BuckleScript.

To manage this, I created a module `typescript.ml` to hold the external declarations.  It looks like this:

```
type sourceFileObject
external createSourceFile :
    string -> string -> int-> bool -> sourceFileObject =
        "createSourceFile" [@@bs.module "typescript"]
```

This is the first time I have used the `@@bs.module` language extension.  There is a good explanation of what it does in the [manual][5], but it basically translates into an import statement and then calling the value of the "external-value" off of the imported module (see the javascript result below.)  Then my `main.ml` looks like this:

```
let () =
    let txt = "declare var foo: number;" in
    let path = "dummy_path" in
    let scriptTarget = 1 in
    let obj = Typescript.createSourceFile path txt scriptTarget false in
    Js.log obj
```

This results in the following JavaScript:

```
var Typescript = require("typescript");

var obj = Typescript.createSourceFile("dummy_path", "declare var foo: number;", 1, /* false */0);

console.log(obj);
```

Notably, the require statement is for the actual "typescript" package, not my typescript module.  The JavaScript emitted for `typescript.ml` is a blank file.  It would probably be a good idea to name the `.ml` file to avoid this ambiguity in the future.  Additionally, the syntax highlighting seems to have some trouble with that language extension, I assume because it has the word `module` which is an OCaml keyword.  I asked about this in the ReasonML discord channels and it turns out that the vim syntax plugin borrows things from Rust.  Rather than fuss around with it too much, I switched to using Atom where the plugin does not have the same issue.  Unfortunately, my 2011 computer makes Atom run a little sluggishly, so we'll see if I keep this up.

One other piece to note here is that the `bool` type in OCaml is encoded as an integer.  To get a JavaScript `boolean`, you need to use `Js.bool` (which I did not do here.)

# What are those arguments?

The code so far makes a call to `createSourceFile`, but I haven't looked up what exactly the arguments mean.  The first argument is the name of the file.  It's not really a mystery what goes here, but what is not immediately clear is why we need a file name when we're passing the contents of the file.  One place it goes is on the `fileName` attribute of the `SourceFileObject` which appears to be just for convenience.  The place where some actual work gets done is that the parser checks the extension of the file to determine if it is one of `.js`, `.jsx`, `.ts` or `.tsx` which modifies its parsing behavior.

The second argument is pretty self explanatory: it's the text we want to parse passed in as a string.

The third argument is called the `ScriptTarget`.  What's not clear is whether it is the desired output of the text, or if it is specifying what format the input is in.  Based on the fact that we're not outputting any JavaScript at this stage, I assume it is specifying the input.  After poking around in the source, it looks like the `languageVersion` controls branches in the `Scanner` to tell whether something counts as an identifier.  Interestingly here, the [tutorial in the repo][6] uses `ES6` as its value which is not in the Enum for ScriptTarget.  I imagine they meant `ES5` or `ES2016`.  I noticed this same mistake in `flowgen`, and that project seems to be fine, so I suppose this flag does not have a big impact.

Finally there is this final `boolean` argument.  From the tutorial linked above, we see that it is `setParentNodes`.  In `flowgen`, it is set to `false`, so I did the same here.  This flag controls whether a function called `fixupParentReferences` gets called.  The comment on that function is :

```
// normally parent references are set during binding. However, for clients that only need
// a syntax tree, and no semantic features, then the binding process is an unnecessary
// overhead.  This functions allows us to set all the parents, without all the expense of
// binding.
```

It seems that we may not need correct parent pointers for our purposes if `flowgen` opted not to call this, however it is also possible that the default is `false` because most callers of this function go through `binding` and therefore have their parents set. I will keep an eye out for that as I proceed.


# Up Next

Next, I want a better understanding of what the `SourceFileObject` offers us before traversing it and attempting to print out OCaml declarations.

The neat thing about this project is that I will be manually converting `.d.ts` files to OCaml modules as I go in order to make calls to the TypeScript Compiler API.  At a certain point, the code I've made so far should be able to help in creating those typings, sort of like a snake eating its own tail.

The code for this blog post is available on [GitHub][7].


[1]:{% post_url 2017-01-14-buckle-pixi %}
[2]:https://github.com/chenglou/reason-js/issues/1
[3]:https://github.com/joarwilk/flowgen
[4]:https://www.typescriptlang.org/docs/handbook/declaration-files/by-example.html
[5]:https://bloomberg.github.io/bucklescript/Manual.html#_binding_to_a_value_from_a_module_bs_module
[6]:https://github.com/Microsoft/TypeScript/wiki/Using-the-Compiler-API
[7]:https://github.com/pcarleton/buckle-typings
