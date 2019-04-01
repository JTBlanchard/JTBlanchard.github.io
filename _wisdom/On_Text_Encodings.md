---
layout: default
title: A brief primer on text encodings
---

As you might expect, text encodings are just conventions for how bits of data represent text.  Encodings will define what numbers represent a character, and may also define how those numbers are represented across multiple bytes.

## Why should I care?

Because when you get text that doesn't match the encoding your program uses or your program expects single bytes to line up with letters, you're going to have a bad time.

## The old encodings

* **ASCII:**  7 bit encoding used in the olden days, representing the English alphabet plus some special characters.  Why 7?  Because in the days before we did error detection in packets, we put parity bits right in the text.  Outside of ancient forms of wireline communication, you'll find ASCII exists as full 8 bits, padded with a leading zero.

* **ISO-8859 series (Latin-1 through Latin-16):** A group of 8 bit encodings that each extend ASCII in a different way, allowing for characters outside the standard English alphabet.  Not all the available space was assigned, but there have been unofficial assignments of unused code in this space, which complicates matters.

* Lots and lots of others.  It is essentially a case of every writing system having its own encoding, that may or may not have maintained compatibility with ASCII.  Moreover, each writing system may have a variety of its own, wildly incompatible encodings (Don't laugh, ASCII users!  We still have EBCDIC running around on IBM mainframes.)

## Enter Unicode

Unicode is not an encoding, but a set of standards about representing text.  Those include the Universal Character Set and a number of encodings.

The Universal Character Set is just a map of integers to characters.  Unicode encodings are different ways of representing those integers.

Without Unicode you have to know what language the source text is in to know what encoding to use.  Without Unicode, you may have text that can't be represented in the standard of another encoding.

## Unicode Encodings

There are a few different Unicode encodings, but in most cases you should only use one of them.  You should know what the others are in order to understand why you generally shouldn't use them.

UCS encodings are the Universal Character Set represented as pure integers.  UCS-2 and UCS-4 are fixed-byte-width encodings, each named according to the number of bytes per character.  UCS-1 is the same integer range, but without a fixed number of bytes per character).

But more often you'll hear about UTF (Unicode Transformation Format).  Their names vary by the minimum number of bits each character will require:  UTF-8, UTF-16, and UTF-32.  UTF-8 and UTF-16 are variable width, and can extend across multiple bytes if the code point doesn't fit in the minimum byte size.  Since part of each UTF-8/16/32 byte is a clue as to the position in a multi-byte string, the Unicode character set won't extend as far as the full 32 bit integer space of UTF-32/UCS-4.

* All Unicode encodings map to the same Universal Character Set, and therefore can be re-encoded between each other without conflict or loss.

* UTF-8 is often used as an interchange format, while operating systems and programming language interpreters/virtual machines may use one of the other UCS or UTF formats for a variety of reasons.

## Why would I use UTF-16 or UTF-32 if UTF-8 is smaller?

There are some very specific circumstances where they might be advantageous:

* UTF-32 and UTF-16 both have variable endianness.  If your text matches the endianness expected by your processor, there can be a very minor speed advantage.  But it probably isn't worth the headache of having to know the CPU architecture of the text source.  (The standard is at least polite enough to specify a Byte Order Mark at the beginning of text to distinguish.)

* UTF-32 may also result in simpler/faster processing for certain specific tasks if it is imperative that you use simple byte counts to identify individual characters.  The same could be said of UTF-16 if you could guarantee that the text used won't exceed two-byte characters (which you probably couldn't).  But any good string processor can divide characters with awareness of variable byte lengths, so why would you ever need to do this?

* UTF-16 may result in smaller storage sizes than UTF-8 for some texts in some languages.  Even though the minimum byte representation for UTF-8 is one byte, some characters that are three bytes in UTF-8 may only be two in UTF-16.  But not all texts have this advantage.

## So what encoding should I use?

**You should use UTF-8.**  Unless you know you will be reading or writing to another encoding, in which case you should still consider converting it in between.

It is a beautiful hack that gives us maximum compatibility in minimum space: 

[![UTF-8 and the Unicode Miracle](https://img.youtube.com/vi/MijmeoH9LT4/0.jpg)](https://www.youtube.com/watch?v=MijmeoH9LT4 "UTF-8 and the Unicode Miracle")

[https://www.youtube.com/watch?v=MijmeoH9LT4](https://www.youtube.com/watch?v=MijmeoH9LT4)

At the level in which most programmers work, what you care about is maximizing compatibility and minimizing storage space.  UTF-8 gives you that excepting in edge cases, and those edge cases may not be worth worrying about.

* UTF-8 is designed to be backwards compatible with ASCII.  Anything that would be valid there is a single byte in UTF-8.  That's probably most of your text.  

* UTF-8 is endian-independent and most text will end up using fewer bytes.  Unless you're writing a string processor for an OS or programming language or you're dealing exclusively in text files composed of mostly three or four byte characters (unlikely), the edge cases aren't going to matter to you. 

* The internet has spoken.  UTF-8 is the standard convention for interchanging data now, unless you're the Chinese government (and even they use a form of Unicode, so it converts cleanly).

* You may think that you'll just use ASCII, but that bombs whenever non-English alphabet text comes into the picture.  And even if you can guarantee that's all you'll need, if something else in your pipeline isn't strict about that you're going to have problems.

* Your old programming language interpreter may use a naïve version of Latin 1 or 8-bit ASCII that blindy reads whole bytes and delivers them to the other end without complaining or modifying them if they don't match known code points, but that's only going to be good enough when you're moving them from point A to point B, in which case you could have just been using plain bytes instead of a string.  When you introduce actual string processing into the picture, things will break if anything doesn't match a code point.

## So why should I even care about other encodings if I should just be using UTF-8?

Because you still have to interact with systems that use other encodings.  

* Windows writes UTF-16 by default with the default system endianness...  and when it writes UTF-8, it includes a byte order mark that your string processing library might need to be aware of in order to ignore.

* Old, non-English text files are a thing.  And some people didn't get the memo.  Many Japanese and Chinese programmers never saw the need to switch.  Maybe that plain text file contains some Spanish accents--that's Latin 1.  Russian is Latin 5.  Good luck.

* Even English-only text can bite you.  Names can contain all kinds of interesting things, and lots of people still preserve diacritical marks in words like *résumé* or *naïve*.

* Sometimes people screw up and don't transcode properly in pipelines that don't catch the error, so you can't always believe what's on the label.  That web app may not have checked before handing it off to the database to write.  That plain-text log file looks like pure ASCII, so it seems safe to read as UTF-8...  until someone used a Euro sign.  That's not legal UTF-8...  but it is a common convention extending Latin 1.  The UTF-8 text processor would throw a fit, but if you'd read it as Latin 1, the encoder would probably have been aware of that.  

* Sometimes data gets corrupted or non-text data gets written to a text file, and those bits may not map to a valid UCS code.

* Some pipelines may read pure bytes instead of strings, or not do proper verification.  It'll be your code that breaks in the end.

## So the lesson is that everything is terrible and there is no solution?

Maybe.  Boiled down, it's this:

* Know your source and destination encodings.
* If you have a choice, that choice should be UTF-8 unless you know it should be something else.
* Validate input strings and the results of transcoding attempts.  What the program thinks the bits are and what the bits actually are may not be accurate if you didn't control the input and the validation.

Yes, it's aggravating.  And yes, you'll still get bitten.  But now you have the awareness to solve as well as avoid encoding problems.
