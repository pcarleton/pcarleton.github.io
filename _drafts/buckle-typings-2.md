---
layout: post
title: "Buckle Typings - Part 2: Walking the AST"
---


In my [last post][1], I layed out the plan to make a converter from TypeScript `.d.ts` type declarations to OCaml external modules useable in BuckleScript.  When I left off, I had made a call to the TypeScript compiler and received a `SourceFileObject`.  In this post, I'll dig into what is in the `SourceFileObject` and how we can use it to traverse the TypeScript AST.


# What is in the `SourceFileObject`?

In order to get a feel for what is in this object, I fired up the node repl and created one in the same way I have been doing so far:

```
$ node
> var tyepscript = require("typescript");
> var sfo = typescript.createSourceFile("dummy_path", "declare var foo: number;", 1, false)
```

I noticed one thing right away.  There's a flag `isDeclarationFile` which is set to `false`.  Setting my "path" to be "dummy.d.ts" instead of "dummy_path" changed this to `true`.

The other things I noticed were that the `nodeCount` was 8.  Understanding what 8 nodes come out of this declaration will be a good next step.


# Why not `utop`?

I asked myself at this point, could I use the OCaml repl instead of the node repl?  Without thinking too hard about it, I fired up `utop` and loaded my `typescript.ml` module:

```
$ utop
utop #  #use "src/typescript.ml";;
...
utop # let sfo = createSourceFile "dummy_path" "declare var foo: number;" 1 false;;
Error: The external function `createSourceFile' is not available
```

This makes sense because we've made external declarations to JavaScript functions.  While in a fully OCaml world, it has no knowledge of where to find JavaScript modules let alone call their functions.

Instead of trying to use JavaScript in the OCaml repl, the more straightforward path is to use to compile the OCaml to JS and use that in the node repl.

# Meet the Nodes

I saw a nodeCount of 8, so I want to know what those are.  When I tried the same call to `createSourceFile` but with the empty string, I got a node count of 2, so I assume the `SourceFileObject` counts as one node.  There's also an `endOfFileToken` field which is populated which I assume is node #2.  In my original declaration, there are 3 nodes, the "kinds" of which are (gotten through `typescript.SyntaxKind[<kind>]`):

* VariableStatement
* DeclareKeyword
* VariableDeclarationList
* VariableDeclaration
* IdentifierObject
* NumberKeyword

When looking at the [TypeScript spec][2] under Ambient Declarations, the names are a little different.  The names expected from that list would be:

* AmbientVariableDeclaration
* AmbientBindingList
* AmbientBinding
* BindingIdentifier
* TypeAnnotation

# Printing in BuckleScript

My next goal is to get enough BS declarations in place to print the list of nodes.  One of the most helpful pieces of that will be encoding the `SyntaxKind` enum.

The BuckleScript manual has [a section][3] a method of using polymorphic variants to model enums.  However, I was having trouble getting this to work.

At first, I thought I could achieve an enum with specific values encoded using the `bs.as` language extension.  I tried this:

```
module SyntaxKind = struct
    type t =
        | EndOfFileToken [@bs.as 1]
        | NumberKeyword [@bs.as 132]
    [@bs.int]
end [@bs]
```

I expected this to result in the `NumberKeyword` being encoded as 132, so if I called `Js.log` on it, I would see 132.  This is not what happened.

I dug a little bit into how the PPX works before deciding to take a step back and think about what I was trying to get out of this enum in order to better understand what the best way forward is.

I want to have several different types of things, I want it to be type-safe, and I want to be able to get "string" representations of those different types.  Since TypeScript encodes these as ints, it looks like I need the following:  int -> SyntaxKind, SyntaxKind -> int and SyntaxKind -> string.  If the `bs.as` keyword worked, the int <-> SyntaxKind relationship would be built in, and I wouldn't need explicit functions for it.

Since that wasn't working quite as I expected, I decided to make my own mapping to the strings and work with a regular variant type.  It would be a little bit more verbose, but would allow for what I want to happen.  Unfortunately, this will make the "typings" have some extra code in them that the original module does not.


At this point, I realized that creating an enum from a TypeScript file is exactly the thing I am trying to automate.  I decided I would try to bootstrap my way through it.

First I wanted to figure out where the enum lived.  Was it fully specified in the `.d.ts` file? Yes! It was.  Next up, what did it look like when parsed?


It's worth considering what I want to get out of t


[1]:
[2]:https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#12.1
[3]:https://bloomberg.github.io/bucklescript/Manual.html#_using_polymorphic_variant_to_model_enums_and_string_types
