---
layout: post
title: "Converting an application to Unicode"
category: Coding
tags: [Unicode, Coding]
---
{% include JB/setup %}

So you need to convert your C/C++ application to [Unicode](http://en.wikipedia.org/wiki/Unicode)?  It's harder than you think.

For many years, there has been one assumption that underlies the biggest problem in modernising code:

 - **One character is equivalent to One Byte**

If the world only spoke English and limited all written communication to the 127 code points enshrined in 7-bit ASCII, this assumption might be reasonable.  Fortunately, Unicode was invented many years ago to solve the problem of having a standardised means for encoding the thousands of written languages used around the world today.

But this means that you can't assume that you can safely take input from the user and treat it simplistically like this:

{%highlight c%}
void printEachCharacter(const char* msg)
{
	unsigned msg_len = strlen(msg);

	for (unsigned i = 0; i < msg_len; i++)
	{
		unsigned char c = msg[i];
		printf("%u: %hhu %c\n", i, c, c);
	}
}
{%endhighlight%}

This will work just fine for 7-bit ASCII.  So if we call this function like:

{%highlight c%}
printEachCharacter("Unicode is great!");
{%endhighlight%}

we will see each letter of the message, one per line as you would expect.  But if the input were taken from a French speaker who likes the taste of , we would see something like this:

	_0:   0x4c   L
	_1:   0x65   e
	_2:   0x20    
	_3:   0x67   g
	_4:   0x6f   o
	_5:   0xc3   ?
	_6:   0xbc   ?
	_7:   0x74   t
	_8:   0x20    
	_9:   0x64   d
	10:   0x65   e
	11:   0x20    
	12:   0x42   B
	13:   0x72   r
	14:   0x61   a
	15:   0x74   t
	16:   0x77   w
	17:   0xc3   ?
	18:   0xb8   ?
	19:   0x72   r
	20:   0x73   s
	21:   0x74   t
	22:   0x20    
	23:   0x65   e
	24:   0x73   s
	25:   0x74   t
	26:   0x20    
	27:   0x67   g
	28:   0xc3   ?
	29:   0xa9   ?
	30:   0x6e   n
	31:   0x69   i
	32:   0x61   a
	33:   0x6c   l
	34:   0x21   !

Various modern development environments such as Java, .NET and Python 3 support Unicode by default, out of the box.  But there are possibly millions of lines of C and C++ code out there which assume that 1 character is stored in 1 byte.  And modernising these codebases is a potentially huge task.  Everwhere you have a `const char*` or `std::string` variable, you have to not only update their types, but you also have to review every function or method and parameter to ensure the

This list is a starting point (but by no means a comprehensive list!) for all of the aspects that need review:

 - Literal strings, messages
 - Localised filenames
 - File I/O
 - Data
 - Network I/O
 - Interfacing with non-Unicode libraries and APIs
 - User input
 - User interface widgets

## Encodings

One of the big decisions you will need to make is *which* flavour of Unicode you're actually going to use.  There's at least three, although there are two primary encodings in widespread use:

 - **UTF8**: one or more bytes per character. Used by Mac OS X, Linux and most Unixen.
 - **UTF16**: two bytes per character. Used by Windows.

(UTF = Universal Transformation Format)

There are several other encodings (such as UCS)

## Briefly - UTF-8

This encoding is probably the easiest to support, as UTF-8 is a superset of the 7-bit ASCII encoding you are most likely already using.  (Apologies to IBMers - I know EBCDIC is still out there too.)  All 7-bit sequences are interpreted as per ASCII.  The 8th bit is used as an escape bit to indicate the sequence is a multi-byte sequence.

## Briefly - UCS-16

ALl recent Windows versions uses this two-byte encoding internally, regardless of...

## Literal strings & messages

Literal strings crop up in many contexts. Whether part of a message to the user, foo baz bar, and so on, 

## Localised filenames

The majority of modern operating systems support extended character sets for naming files in the filesystem.  So there is no good reason to prevent your French users from saving their thoughts on "Pensées à "

## File I/O

When reading and writing data from files, you will invariably be specifying a number of bytes to the file I/O APIs.

## Data

## Network I/O

Just as the Berkeley sockets API for TCP/IP has functions for converting between host and network byte order, you need to ensure that any textual data transmitted over a network channel has the correct encoding and is specified with the correct byte length.

 - Windows: `CreateFile`
 - iOS and OS X: `CFNetwork` supports UTF-8
 - Linux: `send()` and `recv()` on a socket need to use 

## Interfacing with non-Unicode libraries and APIs

## User input
 
## User interface widgets
