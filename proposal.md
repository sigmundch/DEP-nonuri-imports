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
    interpreted as the URI `dart:name`. For example:
```dart
import dart.async;
```
is equivalent to:
```dart
import 'dart:async';
```

  * **default package imports**: a single token identifier imports the default library
    in a package. For example:
```dart
import foo;
```
is equivalent to:
```dart
import 'package:foo/foo.dart'
```

  * **generalized pacakge imports**: other multi-token identifiers that do not
    start with `dart`, are interepreted as a `package:` URL, where each
    identifier is a segment in the path to the file beign imported. For example:
```dart
import foo.src.bar;
```
is equivalent to:
```dart
import 'package:foo/src/bar.dart';
```

## Spec changes

Changes would be similar in the many directives (export, import, part). We
change the `uri` token, for a `uriOrLibraryIdentifier`. For example, imports are
changed from:
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

### An alternative syntax

Use `:` to split the package name from the path. For example:

```dart
import foo:bar;  // equivalent to 'package:foo/bar.dart'
import bar;      // equivalent to 'package:self_package/bar.dart' (where
                 // self_package is the current package name)
import dart:async;
```

### An alternative resolution scheme

Rather than using a set of rules to map our syntax to URIs, we could instead
enlarge the [package spec](https://github.com/lrhn/dep-pkgspec) to map
individual import identifiers to actual URIs.

The advantages is that this could be used outside of the context of packages as
well.

[@sigmundch]: https://github.com/sigmundch
