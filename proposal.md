# Non-Uri Imports

## Contact information

Author: Sigmund Cherem ([@sigmundch][])

[DEP proposal location](https://github.com/sigmundch/DEP-nonuri-imports/blob/master/proposal.md)

## Summary

We propose adding syntax to allow importing libraries in packages using a
compact canonical name, instead of a URI. The name is translated to a URI
internally and has the same semantics as imports do today.

The URI-based syntax continues to be supported. In fact, for some kind of
imports, like relative imports, it continues to be the only way users can
express them.

For additional background, see this [thread in core-dev](https://groups.google.com/a/dartlang.org/forum/#!topic/core-dev/Mtii4OONYkQ).

## Related proposal

This proposal complements another [concurrent proposal][DEP-resolved-part-of]
that addresses library names and part-of directives. In that proposal we discuss
a change in part-of directives to uniquely identify the library they belong to
by using a URI instead of a name.  In essense, such proposal would make Dart
denote files in a consistent way in imports, part, and part-of directives. With
the syntax proposed on our current proposal can help users adopt also the
changes to the part-of directive more easily.

## Syntax and semantics

We propose changing the syntax of the language to allow using a library
identifier instead of a string in import, export, and part directives.

The library identifier is a dotted name, where each segment of the name is used
to denote a portion of the path of the import.

There are two basic rules to construct an import URI from a library identifier:

  * **dart imports**: libraries identifiers of the form `dart.name` are
    interpreted as the URI `dart:name`.
  * **package imports**: dotted names that do not start with `dart` are
    interpreted as a `package:` URL, where the first identifier is the package
    name, and each identifier thereafter is a segment in the path to the file
    being imported.

For convenience, a single identifier `id` is sugar for `id.id`.

The following import examples illustrate the semantics:
```dart
import dart.async;  // is equivalent to: import 'dart:async';
import foo.src.bar; // is equivalent to: import 'package:foo/src/bar.dart';
import foo;         // is equivalent to: import 'package:foo/foo.dart'
```


## Spec changes

We simply change the syntax replacing `uri` for a `uriOrLibraryIdentifier` in
imports, exports, and part directives. For example, imports in Section **18.1**
are changed from:

```dart
importSpecification:  import uri (as identifier)? combinator* ';'
                   |   import uri deferred as identifier combinator* ';'
                   ;
```

to:
```dart
importSpecification:  import uriOrLibraryIdentifier (as identifier)? combinator* ';'
                   |   import uriOrLibraryIdentifier deferred as identifier combinator* ';'
                   ;
uriOrLibraryIdentifier: uri
                      | identifier ('.' identifier)* ';'
                      ;
```

And the text is changed as follows (edit removals are denoted as
~~strikethrough~~, added text is denoted in **bold**):

> An import specivies ~~a URI x~~ where the declaration of an imported library
> is to be found. **This can be a URI x which resolves to the location of the
> imported library, or a library identifier which can be translated as a URI
> according to these rules**:
>  * **an identifier with a single identifier (`id1`) is expanded as
>    an identifier with the same identifier written twice (`id1.id1`).**
>  * **library identifiers of the form `dart.id1`, where `dart` is the first
>    identifier, are equivalent to use the URI `dart:id1`.**
>  * **an identifier `id1.id2...idn` with two or more identifiers, is equivalent
>    to an package import URI where each identifier is a segment in the path:
>    `package:id1/id2/.../idn.dart`.**

The same change is needed in the section **18.2** describing exports,
and **18.3** describing parts.

## Alternatives

Some other alternatives we've considered.

### The 'name:path' syntax

Under this syntax the package name is explicit. If not given, it defaults to the
current package. We use `:` to split the package name from the path. For
example:

```dart
import foo;          // equivalent to import 'package:foo/foo.dart'
import foo:src.bar;  // equivalent to import 'package:foo/src/bar.dart'
import :src.bar;     // equivalent to import 'package:self_package/src/bar.dart'
                     // (where self_package is the current package name)
import dart:async;   // same as import 'dart:async'
```

### The lookup resolution approach

Rather than using rules to translate our syntax to URIs, we would instead
add to the [package spec](https://github.com/lrhn/dep-pkgspec) a map
of individual import identifiers to actual URIs.

The advantages of this alternative are:
* this decouples library names from library locations.
* the new import syntax could be used for normal imports, not just in the
  context of a package.

The main disadvantages is that the import structure may not correspond naturally
to the layout of the code. So we would need to rely on conventions and style
guides to keep the ecosystem clear and homogeneous.

### Alternative spec modification

Instead of specifying our changes in each section describing imports, exports,
and parts. We could instead modify the spec in describing URIs in Section
**18.5**. The proposed changes would be:

```dart
uri:
   stringLiteral
  | identifier (. identifier)*
```

Where:
  * `id` is sugar for `id.id`, which is sugar for `'package:id/id.dart'`
  * `dart.id` is sugar for `'dart:id'`
  * `id1.id2.id3` is sugar for `package:id1/id2/id3.dart`.

[DEP-resolved-part-of]: https://github.com/sigmundch/DEP-resolved-part-of/blob/master/proposal.md
[@sigmundch]: https://github.com/sigmundch
