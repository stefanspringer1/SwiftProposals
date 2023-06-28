# Extending optional chains to include for loops

* Proposal: [SE-NNNN](NNNN-filename.md)
* Authors: [Stefan Springer](https://github.com/stefanspringer1/), [Author 2](https://github.com/swiftdev)
* Review Manager: TBD
* Status: **Awaiting implementation**

*During the review process, add the following fields as needed:*

* Implementation: [apple/swift#NNNNN](https://github.com/apple/swift/pull/NNNNN) or [apple/swift-evolution-staging#NNNNN](https://github.com/apple/swift-evolution-staging/pull/NNNNN)
* Decision Notes: [Rationale](https://forums.swift.org/) (TODO), [Additional Commentary](https://forums.swift.org/) (TODO)
* Bugs: –
* Previous Revision: [1](https://github.com/apple/swift-evolution/blob/...commit-ID.../proposals/NNNN-filename.md)
* Previous Proposal: [SE-0231](https://github.com/apple/swift-evolution/blob/main/proposals/0231-optional-iteration.md)

## Introduction

For-in loops only work with non-optional sequences. An optional sequence is not allowed as the sequence expression of a for-in loop. It may be convenient – as also being discussed in the rejected proposal [SE-0231](https://github.com/apple/swift-evolution/blob/main/proposals/0231-optional-iteration.md) – to allow optional sequences in for-in loops, maybe being annotated with an appropriate question mark signaling that an optional sequence is used at this point. The solution in the proposal SE-0231 was using `for?` in the case of an optional sequences and that specific solution [has been rejected](https://forums.swift.org/t/rejected-se-0231-optional-iteration/17805), but while doing so an alternative solution has been mentioned. _(Just to be sure that there is no misunderstanding for the reader at this point: The `for?` formulation is not (!) part of this proposal.)_

The solution in this proposal follows the alternative solution, allowing the same sequence term (which could contain a trailing question mark, see the section “Proposed solution” below for a definition) as could be written in front of the dot of a method call. From the [rejection of SE-0231](https://forums.swift.org/t/rejected-se-0231-optional-iteration/17805):

```Swift
let optionalSequence: [Int]?
let optional: (sequence: [Int], otherStuff: Int)?

for x in optionalSequence? { ... }
for x in optional?.sequence { ... }
```

This means that when switching between the for-in loop and an application of `forEach`, this term would not have to be changed. The optionality is always clear from a question mark in the sequence term. While motivations described in the old, rejected proposal might still be valid, we try a fresh take in the motivation section below, as the proposed solution differs from the rejected one.

This proposal draft has been presented in [this forums topic](https://forums.swift.org/t/pitch-extending-optional-chains-to-include-for-loops/65848). Before, some arguments in favour or against this solution have already been discussed in [another forums topic](https://forums.swift.org/t/still-or-again-interest-in-optional-iteration/65730).

## Motivation

In the following code, we want to iterate through the children of the first child with name “status” of an XML element (using [this library](https://github.com/stefanspringer1/SwiftXML), which is an appropriate real use case, but the example should be very clear and also easily generalizable):

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

From this code is is less clear from a first quick view which the elements are that we iterate through. This proposal would make the other, simpler looking code valid.

Note that there might be even more complex use cases, where the lack of the proposed solution would lead to even more nested `if let` and for-in loops. (The use case of complex XML querying and manipulation is a good example, as it demands concise formulations in order for the code to be still easily understandable.)

One might argue that optional chaining is not necessary here, because you _can_ use the according `if let` expressions — but note that the same argument could be used against optional chaining at some other places. So even if one does not see a big improvement for for-in loops, making the language feel more consistent (“We have this great feature of optional chaining, but why can’t I use it at this place?”) could be an argument in favour of the proposed language change (also see the discussion of alternative 2 below). So it could be seen less as new language feature _per se_ but as part of a “streamlining” of the language.

Optionality is then always clear from a question mark in the sequence term. The example above has a `?`  in the chain, and e.g when the function call `giveMeAnOptionalSequence()`  returns an optional sequence and you would like to iterate through the sequence if it exists, you would have to write:

```Swift
for item in giveMeAnOptionalSequence()? { … }
```

From the rejected proposal [SE-0231](https://github.com/apple/swift-evolution/blob/main/proposals/0231-optional-iteration.md):

>While usage of optional sequences is often treated as misconception, there are several common ways one could end up with an optional sequence through Standard Library APIs and language constructs themselves. Amongst the most prevalent are optional chaining and dictionary getters. An indentation-sensitive area of which optional arrays are an integral part is decoding and deserialization, i.e parsing a JSON response. Swift currently doesn't offer a mechanism for expressing optional iteration directly: optional sequences are illegal as a for-in loop argument. For a safe option, developers often resort to optional binding, which requires additional nesting ...

## Proposed solution

The proposed solution is that when writing a for-in loop:

```Swift
for … in <sequence term> { … }
```

then the allowed `<sequence term>`  would be the same as in the case of the method application for a sequence, e.g. when using `forEach`:

```Swift
<sequence term>.forEach { … }
```

Such a `<sequence term>` could include a trailing question mark.

As part of the solution that we propose, the following code in the above example would be valid:

```Swift
for element in myElement.children("status").first?.children { … }
```

As already explained in the motivation section, this code is cleaner than the one that ensures a non-optional sequence by adding an `if let` statement: it is more easily understandable what the things are that we want to iterate through.

At this point we answer possible objections against the proposed solution. Another objection could be the existence of a suitable alternative, see the section about the alternatives further below.

### Objections from the rejection of proposal SE-0231

The following objections are from the [rejection of proposal SE-0231](https://forums.swift.org/t/rejected-se-0231-optional-iteration/17805):

>There are nonetheless legitimate concerns to address with extending optional chaining to for loops. Although it extends an existing language feature, it does so in a non-obvious and potentially confusing way. It could also be seen as establishing a "slippery slope" to creeping optional chaining into other contexts. Although optional chaining generally composes well, it still gets ugly in some cases, particularly if one were to use the `try?` operator in a for loop:
>
>```
>for x in (try? operationThatMightThrow())? { ... }
>```
>
>In response to that last point, one could point to the precedent recently set by [SE-0230, flattening the nested optionals produced by `try?` ](https://forums.swift.org/t/se-0230-flatten-nested-optionals-resulting-from-try/16570), and argue that `try?` effectively begins an optional chain of its own.

As for the possible “ugliness” of optional chaining, one could point out that no more ugliness is being introduced than existed before, so to speak – but see the objection in the following section. Note that the mentioned proposal [SE-0230](https://github.com/apple/swift-evolution/blob/main/proposals/0230-flatten-optional-try.md) has already been implemented for Swift 5.

As for the “non-obvious way” of introducing non-optional sequences, this might be true to some degree, but not more than in some other cases of optional chaining, and the proposed solution is still quite explicit in comparison to allowing optional sequences in for-in loops without making this explicit (see alternative 5 below).

### Objection to the trailing question mark in an isolated term

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

The following alternatives have been considered. The first three alternatives use features of the current Swift language and do not demand any change to the language or its standard libraries.

### Alternative 1: Use `if let` to ensure a non-optional sequence

This has already been discussed above.

### Alternative 2: Use `forEach` instead

A for-in loop and `forEach` have different semantics, e.g. you might want to use a for-in loop to be able to stop the iteration (see the example in the motivation). It should be possible to switch between those two loop options according to your needs without being “penalized” with a more complex formulation in one case.

### Alternative 3: Using empty sequences as a fallback

This is another option without any change to the Swift language.

Use the following code instead:

```Swift
for element in myElement.children("status").first?.children ?? EmptyElementSequence() {  }
```

where `EmptyElementSequence()` is an appropriate empty sequence.

This would have the drawback of having to introduce a sequence (the empty sequence) that we actually are not interested in, when all we want is that nothing happens if the optional sequence is `nil`.

### Alternative 4: Add a property of the optional sequence to ensure the usage of a non-optional sequence

This is a variation of alternative 3:

```Swift
for element in myElement.children("status").first?.children.sequence {  }
```

The new `sequence` property (one might want to choose a different name) of an optional sequence would give you the unwrapped sequence if the sequence exists, and else an approporiate empty sequence.

We think that this does not feel as a natural expression of you intent (it feels like getting a sequence from a sequence, compare it when using it before `forEach`), and it dispenses with the described symmetry between the for-in loop and `forEach` which we believe is a good thing. It is also not easy to discover as a feature; such an argument of discoverability [was part of the rejection of the `for?` solution](https://forums.swift.org/t/rejected-se-0231-optional-iteration/17805).

An argument in favour of this alternative could be that the language itself would not have to be changed, but only the according library. But this is still a change in the Swift distribution – we think that adding an according extension in your own library is not a good solution, because this way you stray from the usual path of how for-in loops are used.

### Alternative 5: Allowing optional sequences in for-in loops without making it explicit

For any optional sequence `mySequence` one could allow:

```Swift
for item in mySequence { … }
```

Even if no “logical” arguments could be found against this option, such a solution would be at least controversial (see e.g. [this comment](https://forums.swift.org/t/still-or-again-interest-in-optional-iteration/65730/5)). The argument against it is that one would like to see if the sequence in the for-in loop is optional or not. With the solution in this proposal, one could always recognize the optionality of the sequence.

### Alternative 6: Use `for?`

This is the proposed solution from the rejected proposal [SE-0231](https://github.com/apple/swift-evolution/blob/main/proposals/0231-optional-iteration.md) and is only listed here for completeness (cf. [the rejection](https://forums.swift.org/t/rejected-se-0231-optional-iteration/17805)).

## Acknowledgments

The title “Extending optional chains to include for loops” is taken from the [rejection of proposal SE-0231](https://forums.swift.org/t/rejected-se-0231-optional-iteration/17805).

The discussion in the aforementioned [forums topic](https://forums.swift.org/t/still-or-again-interest-in-optional-iteration/65730) was very helpful.
