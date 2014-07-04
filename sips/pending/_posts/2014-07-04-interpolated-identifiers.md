
---
layout: sip
disqus: true
title: SIP-24 Literals and Identifiers Eligible for Simple String Interpolation
---

**Kevin Wright**

## Motivation ##

The general form of an expression interpolated into a string is:

    s"Bob is ${ user.age } years old."

If the expression is an identifier, then a simpler form is allowed:

    s"Bob is $age years old."

However, the identifier is restricted to alphanumeric characters and underscore in SIP-11.

Any other identifiers must be interpolated using the general form:

    scala> val age = 42
    age: Int = 42

    scala> val age_ = 43
    age_: Int = 43

    scala> val age_? = 44
    age_?: Int = 44

    scala> val ***! = 45
    ***!: Int = 45

    scala> f"$age_"
    res0: String = 43

    scala> f"$age_?"
    res1: String = 43?

    scala> f"$age?"
    res2: String = 42?

    scala> f"${age_?}"
    res3: String = 44

    scala> f"${***!}"
    res4: String = 45

This is also true for backquoted identifiers, which are of special interest
in domain-specific languages:

    scala> val n = 22
    scala> val `a..n` = 'a' until ('a'+n) mkString ", "
    scala> s"implicit def hlistTupler$arity[${`A..N`}] : Aux[${`A::N`}, ${`(A..N)`}] = ..."

There is a striking gain in clarity when the backquoted identifiers are
expressed in simple form:

    sip24> s"implicit def hlistTupler$arity[$`A..N`] : Aux[$`A::N`, $`(A..N)`] = ..."

Braces are used as simple delimiters:

    scala> val who = "some"
    who: String = some

    scala> s"I coulda been ${who}body."
    res5: String = I coulda been somebody.

Compare:

    sip24> s"I coulda been $`who`body."

Compare the post-fix expression:

    scala> `who`length
    res6: Int = 4

Another expression that benefits from lifting into the simpler syntax is
the literal quote:

    scala> s"I coulda been ${'"'}${who}body.${'"'}"
    res7: String = I coulda been "somebody."

Expression syntax is required because no escape characters are interpreted
when parsing processed string literals:

    scala> s"I coulda been \"${who}body.\""
    <console>:1: error: ';' expected but '.' found.
           s"I coulda been \"${who}body.\""
                                       ^

There is consensus that `$"` would serve well as an escape for the literal
quote, by analogy to the literal dollar `$$`:

    sip24> s"I coulda been $"${who}body.$""

## Proposal ##

All valid Scala identifiers should be eligible for simple interpolation,
including backquoted identifiers and identifiers with operator characters.

Since the longest valid identifier is used, this proposal changes the interpretation
of processed strings that contain identifiers with operator characters.
Compare examples res1 and res3 above.

Since the escape `` $` `` is currently disallowed in interpolated strings,
adding the facility for backquoted identifiers is compatible with current usage.

## Syntax changes ##

This proposal requires the following change to the syntax for
escapes within processed strings:

    escape   ::= ‘$$’ 
              |  ‘$"’
              |  ‘$’ id
              |  ‘$’ BlockExpr

An escape character for literal quote is added, and the escape for
simple interpolation is relaxed to include any valid identifier.
