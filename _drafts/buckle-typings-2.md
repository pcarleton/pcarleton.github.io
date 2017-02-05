---
layout: post
title: "Buckle Typings - Part 2: Walking the AST"
---


In my [last post][1], I layed out the plan to make a converter from TypeScript `.d.ts` type declarations to OCaml external modules useable in BuckleScript.  When I left off, I had made a call to the TypeScript compiler and received a `SourceFileObject`.  In this post, I'll dig into what is in the `SourceFileObject` and go through typing an Enum.


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

Instead of trying to use JavaScript in the OCaml repl, the more straightforward path is to compile OCaml to JS and use that in the node repl.

# Meet the Nodes

I saw a nodeCount of 8, so I wanted to know what those nodes were.  When I tried the same call to `createSourceFile` but with the empty string, I got a node count of 2.  There is an `endOfFileToken` field which is populated which I assume is one node, and the other node I assume is the "root" aka the `SourceFileObject` itself.  In my original declaration, there are 6 nodes, the "kinds" of which are (gotten through `typescript.SyntaxKind[<kind>]`):

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


# Encoding an enum

After seeing the tags for a very simple declaration, I realized that an OCaml module for SyntaxKind would be useful in order to branch on what type of node I see.  I started doing this manually before catching myself: This was the reason I started this project in the first place!

Things got a little strange as I started thinking about how this project would start consuming its own output before it was complete.  I am currently reading Godel Escher Bach and it reminded me of a "strange loop".

I decided to encode the TypeScript enum type as a module with a variant type and some functions for getting names from that variant type.  Now technically, this enum type is not the same as creating an interface to the actual "Enum" object in TypeScript, but a variant seemed more user friendly than creating an object with a ton of methods that you would access with `##`.  Basically what I was looking to create was this:

```
module EnumType = struct
    type t =
      | EnumVal1
      | EnumVal2

    let getName : t -> string
end
```

Fortunately, enums are encoded as ints in the order they are declared in BuckleScript.  This is relying on an implementation detail of BuckleScript with is generally a no-no, but I decided it was worth it at this point.  One issue that arises is that TypeScript allows declaring what number an enum value corresponds to.  I briefly tried using the `[@@bs.as]` language extension as explained in the [polymorphic variants section][3], but it did not result in the variant being encoded as different values.  I decided to ignore this for now.

The `getName` function is a little tricky because I needed the resulting JavaScript to be `typescript.SyntaxKind[<kind>]`.  This looks like a time to use the `[@@bs.get_index]` language extension.  However, that extension requires an object as its first argument, so it didn't look like I could do it in a single `external` declaration.  I managed to get it to work with 4 declarations:

```
type rawEnum
external enum : rawEnum = "SyntaxKind" [@@bs.module "typescript"] [@@bs.val]
external _getName : rawEnum -> t -> string = "" [@@bs.get_index]
let getName tv = _getName enum tv
```

I declare the type for the raw enum. Next, I bind the actual Enum object to a variable with that type.  Then, I declare the external function that takes a raw enum and my type `t` and returns the string value of that enum.  Finally, I make the `getName` function call my externally bound function with the externally bound value as its first argument.

# Programattically Generating the Encoding

Now that I had settled on the encoding, my next step was to generate it with code for the actual SyntaxKind enum. This essentially amounted to building strings that match the encoding I came up with while swapping in values from the actual Nodes.  One piece I had to add was proper nesting for indentation.  To achieve this, I used the following submodule:

```
module Line = struct
    type t = { text: string; level : int}

    let nest n v =
        {text=v; level=n}

    let indent = "    "

    let toVal : t -> string = fun l ->
        let spaces = string_repeat indent l.level  in
            spaces ^ l.text
end
```

This was a pretty crude way of keeping track of the nesting level of individual lines of text.  I think handling nesting on a more meaningful layer than line of text would be useful, but this was good enough to get things going for now.

The full code is available on [GitHub][4].


[1]:{% post_url 2017-01-20-buckle-typings %}
[2]:https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#12.1
[3]:https://bloomberg.github.io/bucklescript/Manual.html#_using_polymorphic_variant_to_model_enums_and_string_types
[4]:https://github.com/pcarleton/buckle-typings/blob/54f66518b4077f003389f7d6d148f690e7bbb6b2/src/main.ml
