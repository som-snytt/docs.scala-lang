---
layout: sip
disqus: true
title: SIP-26 - Intuitive Interpolation Escapes
---

__A. P. Marki__

__first submitted 8 August 2015__

## Motivation ##

String interpolation is a popular feature. Everyone wants to write s"hello, \"$world\"" but they can't.

Currently, the parser takes an interpolated string as arbitrary characters between double quotes (`"`).

Interpolated arguments are introduced by dollar sign (`$`) and the dollar sign itself can be escaped by
doubling it (`$$`).

The current parser is agnostic about backslash as an escape, and it is up to the interpolator to
determine the semantics of backslash escapes and whether escapes are supported at all.

Literal double quote is supported by interpolated expressions:

    s"hello, ${ '"' }$world${ '"' }"

It has been informally proposed to support literal quote with a dollar escape:

    s"hello, $"$world$""

A formatting interpolator can define its own convention:

    F"hello, %q$world%s%q"

These alternatives are not as intuitive as the standard escape for double quote as supported for
string literals.

## Syntax ##

The proposed syntax is that the parser will honor backslash escapes for `$`, `"` and `\`.

Since the parser does not translate any escapes, the semantics of these escapes are defined by
the interpolator.

    s"\$s to \"do\\nuts\""

can be expressed currently:

    s"$$s to ${'"'}do\\nuts${'"'}"

The backslash is escaped because the `s`-interpolator processes standard escapes.

Under this proposal, `\$` is also processed by the `s`-interpolator.

For uniformity, dollar escapes are proposed for each of `$$`, `$"` and `$\`.

    raw"$\$$"     // "\\$"

This is the same as

    raw"\$"

since the parser passes the escaped text `\$` to the `raw`-interpolator, which ignores escapes.

But because the parser will honor the escaped dollar, and the `raw`-interpolator abjures all escapes,

    val x = "X"
    raw"\$x"        // "\$x"
    raw"$\$x"       // "\X"

To support trailing backslash in the interpolated string, an escaped double quote is taken as
a literal backslash and terminal quote if it is the last double quote on the line.

    s"C:\Program Files\"

More precisely, the parser will locate the end of an ordinary (not triple-quoted) interpolated string
by finding the next unescaped quote (or the last escaped quote on the line if all remaining quotes are escaped).

### Examples

{% highlight scala %}
    val x = "X"
    val rx =r"\Q$x\E\\\$"
    "X\\" match { case rx() => true }
{% endhighlight %}

## Restrictions ##

Previously,

  raw"hello, \" contains "hell"

would return true due to quoting hell.

Under this proposal, the third quote character would terminate the interpolated string and the remaining text
would not parse because the last quote starts a string literal that is not terminated.

## See Also ##

