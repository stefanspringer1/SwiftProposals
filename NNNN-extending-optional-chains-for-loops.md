# Extending optional chains to include for loops

* Proposal: [SE-NNNN](NNNN-filename.md)
* Authors: [Author 1](https://github.com/swiftdev), [Author 2](https://github.com/swiftdev)
* Review Manager: TBD
* Status: **Awaiting implementation**

*During the review process, add the following fields as needed:*

* Implementation: [apple/swift#NNNNN](https://github.com/apple/swift/pull/NNNNN) or [apple/swift-evolution-staging#NNNNN](https://github.com/apple/swift-evolution-staging/pull/NNNNN)
* Decision Notes: [Rationale](https://forums.swift.org/), [Additional Commentary](https://forums.swift.org/)
* Bugs: [SR-NNNN](https://bugs.swift.org/browse/SR-NNNN), [SR-MMMM](https://bugs.swift.org/browse/SR-MMMM)
* Previous Revision: [1](https://github.com/apple/swift-evolution/blob/...commit-ID.../proposals/NNNN-filename.md)
* Previous Proposal: [SE-XXXX](XXXX-filename.md)

## Introduction

For-in loops only work with non-optional sequences. An optional sequence is not allowed as the sequence expression of a for-in loop. It may be convenient – as also being discussed in the rejected proposal [SE-0231](https://github.com/apple/swift-evolution/blob/main/proposals/0231-optional-iteration.md) – to allow optional sequences in for-in loops, maybe being annotated with an appropriate question mark signaling that an optional sequence is used at this point. The solution in the proposal SE-0231 was using `for?` in the case of an optional sequences and that specific solution [has been rejected](https://forums.swift.org/t/rejected-se-0231-optional-iteration/17805), but while doing so an alternative solution has been mentioned.

The solution in this proposal follows this alternative solution, which is kind of a more general solution in comparison to the rejected one, allowing the same sequence term (which could contain a trailing question mark) as could be written in front of the dot of a method call. This means that when switching between the for-in loop and an application of `forEach`, this term would not have to be changed. While motivations described in the old, rejected proposal might still be valid, we try a fresh take in the motivation section below, as the proposed solution differs from the rejected one.

Some arguments for and against this solution have already been discussed in a [forums topic](https://forums.swift.org/t/still-or-again-interest-in-optional-iteration/65730).

This proposal draft has been presented in [this forums topic](https://forums.swift.org/t/XXX).

## Motivation

In the following code, we want to iterate through the children of the first child with name "status" of an XML element (using [this library](https://github.com/stefanspringer1/SwiftXML), but the example should be very clear and also easily generalizable):

```Swift
myElement.children("status").first?.children.forEach { element in … }
```

Now let us assume that we would like to stop the iteration when we found a specific element. A for-in loop would then be more appropriate, and after a first refactoring step we may have written the following:

```Swift
for element in myElement.children("status").first?.children { … }
```

Because `myElement.children("status").first?.children` is an optional sequence, this code is not valid. The canonical way to resolve this is to use to the following code:

```Swift
if let firstStatus = myElement.children("status").first {
    for element in firstStatus.children { … }
}
```

From this code is is less clear from a first quick view which the elements are that we iterate through.

## Proposed solution

The proposed solution is that when writing a for-in loop:

```Swift
for … in <sequence term> { … }
```

then the allowed `<sequence term>`  would be the same as in the case of the method application for a sequence, e.g. when using `forEach`:

```Swift
<sequence term>.forEach { … }
```

(Note that such a `<sequence term>` could include a trailing question mark.)

As part of the solution that we propose, the following code in the above example would be valid:

```Swift
for element in myElement.children("status").first?.children { … }
```

As already explained in the motivation section, this code is cleaner than the current possible alternative when using a for-in loop: it is more easily understandable what the things are that we want to iterate through.

### Objections from the rejection of proposal SE-0231

The following objections are from the [rejection of proposal SE-0231](https://forums.swift.org/t/rejected-se-0231-optional-iteration/17805):

>There are nonetheless legitimate concerns to address with extending optional chaining to for loops. Although it extends an existing language feature, it does so in a non-obvious and potentially confusing way. It could also be seen as establishing a "slippery slope" to creeping optional chaining into other contexts. Although optional chaining generally composes well, it still gets ugly in some cases, particularly if one were to use the `try?` operator in a for loop:
>
>```
>for x in (try? operationThatMightThrow())? { ... }
>```
>
>In response to that last point, one could point to the precedent recently set by [SE-0230, flattening the nested optionals produced by `try?` ](https://forums.swift.org/t/se-0230-flatten-nested-optionals-resulting-from-try/16570), and argue that `try?` effectively begins an optional chain of its own.

As for the possible “ugliness” of optional chaining, one could point out that no more ugliness is being introduced than existed before, so to speak – but see the objection in the following section. Note that the mentioned [proposal SE-0230](https://github.com/apple/swift-evolution/blob/main/proposals/0230-flatten-optional-try.md) has already been implemented for Swift 5.

As for the “non-obvious way” of introducing non-optional sequences, this might be true to some degree, but the proposed solution is still quite explicit in comparison to allowing optional sequences in for-in loops withoput making this explicit (see one of the alternatives listed below).

### Objection to the trailing question mark in a isolated term

This proposal would bring more “isolated” terms with a trailing question mark (i.e. the question mark is is not followed by a method or index application) which are quite unsual in Swift and might even be considered as anti-pattern. E.g. the following is valid Swift code, but a kind of an unusual one, and people might object of better not using it:

```Swift
myVariable? = myValue
```

One might argue that in the case of a for-in loop:

```Swift
for item in myOptionalSequence? { … }
```

the question mark could be read as a simple reminder that the sequence might be empty and that people will get used to this notation.

## Detailed design

TODO: _Describe the design of the solution in detail. If it involves new
syntax in the language, show the additions and changes to the Swift
grammar. If it's a new API, show the full API and its documentation
comments detailing what it does. The detail in this section should be
sufficient for someone who is *not* one of the authors to be able to
reasonably implement the feature._

## Source compatibility

Existing code would still be valid and behave the same.

## Effect on ABI stability

The proposed solution would not change the existing ABI.

## Effect on API resilience

An existing API will not change.

## Alternatives considered

The following alternatives have been considered:

### Alternative 1: Use `if let` to ensure a non-optional sequence

This has already been discussed above.

### Alternative 2: Using empty sequences as a fallback

This is another option without any change to the Swift language:

Use the following code instead:

```Swift
for element in myElement.children("status").first?.children ?? EmptyElementSequence() {  }
```

where `EmptyElementSequence()` is an appropriate empty sequence.

This would have the drawback of having to introduce a sequence (the empty sequence) that we actually are not interested in, when all we want is that nothing happens if the optional sequence is `nil`.

### Alternative 3: Allowing optional sequences in for-in loops without making it explicit

For any optional sequence `mySequence` one could allow:

```Swift
for item in mySequence { … }
```

Even if no “logical” arguments could be found against this option, such a solution would be at least controversial (see e.g. [this comment](https://forums.swift.org/t/still-or-again-interest-in-optional-iteration/65730/5)). The argument against it is that one would like to see if the sequence in the for-in loop is optional or not. With the solution in this proposal, one could always recognize the optionality of the sequence.

## Acknowledgments

The title “Extending optional chains to include for loops” was taken from the [rejection of proposal SE-0231](https://forums.swift.org/t/rejected-se-0231-optional-iteration/17805).

The discussion in the aforementioned [forums topic](https://forums.swift.org/t/still-or-again-interest-in-optional-iteration/65730) was very helpful.