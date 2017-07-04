Regular Expression to Remove Comments
---

## Problem

This was developed in response to a question on Stack Overflow, [Removing comments using regex](https://stackoverflow.com/q/42287216/7412956). My answer on Stack Overflow is [here](https://stackoverflow.com/a/42437107/7412956).

What the OP needed was a regular expression to use in C# to remove comnents from lines of a file. All comments were specified as beginning with a double slash `//` and everything after that is ignored. Comments may also begin at the start of a line, resulting in the line being blank. Quotes, double or single, when properly closed, will _shield_ any double slashes from triggering a comment.

## Solution

I have tested this pattern with every pathological example I can think of, including

- Lines that contain what I believe to be syntax errors
- A quoted string that has too many quotes
- A quoted string that has too quotes
- A quoted string that has a double escaped quote, which is, therefore, _not_ escaped
- With quoted strings in the comments, which I have been known to do when I want to remind myself of alternatives.

The patterns given here are as they need to be seen by the regex engine. I am _not_ a C# programmer, so I don't know all the nuances of escaping quoted strings. Getting the pattern into your code, such that all the backslashes and quotes are seen properly by the regex engine is still up to you. Maybe C# has the equivalent of Perl's `heredoc` syntax.

One version `slash_comment_stripper_single` is a one-liner, which does the job, and is only 148 bytes. The other version `slash_comment_stripper_multi` is a multi-line that is the same regex pattern to be used with the _ignore pattern whitespace_ option turned on. This version is heavily commented. This allows for your code to explain this monstrosity so that when someone else looks at it, or in a few months when you have to work on it yourself, there's no _WTF_ moment. I think the comments explain it well, but feel free to change them any way you please.

## Explanation

Taking advantage of the C# environment given by the problem enabled the use of conditional matching.

Conditional matching is a grouping construct that has varying level of support in regex engines. I believe as used here, with a lookaround, it's only the `.NET` and `PCRE` engines that will accept it **YMMV**. It is a tertiary type: `(?(_condition_)_then_|_else_)`. The `_condition_` pattern is treated as a zero-width assertion. If the assertion pattern matches, the `_then_` pattern is used in an attempted match, otherwise the `_else_` pattern is used in an attempted match. Without this construct, the pattern was growing to uncommon lengths, and was still failing on some of my pathological test cases.

When used the pattern will return the non-comment portion of the line(s). The pattern has a newline `\n` in it to allow for applying it to an entire file. You may need to modify that if you system interprets newlines in some other fashion, for example as `\r` or `\r\n`. To not use it in single line version you can remove that if you choose. It is at characters 17 and 18 in the one-liner and is on the fifth line, 6th and 7th printing characters in the multi-line version. You can safely leave it there, however, as in single-line it makes no difference, and in multi-line mode it will return a newline for lines of code that are either blank, or have a comment beginning in the first column. That will keep the line numbers the same in the original version and the stipped version if you write the results to a new file. Makes comparison easy.

The only time that it trips up is if there is a double slash inside a _seemingly_ quoted string and somehow that string is malformed and the double slash ends up legally outside the properly quoted portion. Syntactically that makes it a valid comment, even though not the programmer's intention. So, from the programmer's perspective it's wrong, but by the rules, it really is a comment. Meaning, the pattern only appears to trip up.

As mentioned above, the conditional match grouping has limited support. It will fail in many of the online regular expression testing sites because of the conditional construct.  I choose to do my testing in the [.NET Regex Tester](http://regexstorm.net/tester), which can handle those constructs. It includes a nice Reference as well. Given the proper selections on the side, you can test either version above, and experiment with it as well. 

