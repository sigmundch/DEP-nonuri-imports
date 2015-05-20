# Non-Uri Package Imports

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

There are three rules to construct an import URI from a library identifier:

  * **dart imports**: libraries identifiers of the form `dart.name` are
    interpreted as the URI `dart:name`.
  * **pacakge imports**: dotted names that do not start with `dart` are
    interepreted as a `package:` URL, where the first identifier is the package
    name, and each identifier thereafter is a segment in the path to the file
    beign imported.
  * **default package imports**: for short, a single identifier corresponds to
    the default library in a package with that name.

The three imports below illustrates each of these rules respectively:
```dart
import dart.async;  // is equivalent to: import 'dart:async';
import foo.src.bar; // is equivalent to: import 'package:foo/src/bar.dart';
import foo;         // is equivalent to: import 'package:foo/foo.dart'
```


## Spec changes

We simply change the syntax replacing `uri` for a `uriOrLibraryIdentifier` in
imports, exports, and part directives. For example, imports are changed from:
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

## Alternatives

Some other alternatives we've considered.

### The 'name:path' syntax

Under this syntax the package name is explicit. If not given, it defaults to the
current package. We use `:` to split the package name from the path. For
example:

```dart
import foo:src.bar;  // equivalent to import 'package:foo/src/bar.dart'
import src.bar;      // equivalent to import 'package:self_package/src/bar.dart'
                     // (where self_package is the current package name)
import dart:async;   // same as import 'dart:async'
```

### The lookup resolution approach

Rather than using rules to translate our syntax to URIs, we would instead
add to the [package spec](https://github.com/lrhn/dep-pkgspec) a map
of individual import identifiers to actual URIs.

The advantages of this alternative is that this could be used outside of the
context of packages as well.

[DEP-resolved-part-of]: https://github.com/sigmundch/DEP-resovled-part-of/blob/master/proposal.md
[@sigmundch]: https://github.com/sigmundch
