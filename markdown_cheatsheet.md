Markdown Cheatsheet
===================

See [Daring Fireball: Markdown syntax](http://daringfireball.net/projects/markdown/syntax) for the full description.

[Github](https://github.com) uses its own version of markdown, described at
[github flavored markdown](http://github.github.com/github-flavored-markdown/)

Headings
--------

Underline with `=` to generate level 1 headings, with `-` for level 2.

Alternatively enclose text with `###` for level 3 etc (number of \# indicates
level, from 1 to 6), e.g.

    #### Level 4 heading #### 

Emphasis
--------

The markdown:

    This is _emphasised_; so is *this*

generates: "This is _emphasised_; so is *this*"

    This is __very emphasised__, as is **this**

generates: "This is __very emphasised__, as is **this**"

Lists
-----

### Bulleted (unordered) lists ###

Use `*`, `+` or `-` to indicate bullets in a list. So this:

    * Unos
    * Dos
    * Tres

produces this:

* Unos
* Dos
* Tres

### Numbered lists ###

Use a number followed by a period e.g. `1.` to indicate list items, for
example:

    1. Un
    2. Deux
    3. Trois

produces:

1. Un
2. Deux
3. Trois

Note that the numbers in the markdown don't have to be consecutive,
however they will be consecutive in the generated HTML.

Blockquotes
-----------

Use email-style `>` to create a blockquote, e.g.

    > This is blockquoted text
    >
    > which ends here

generates:

> This is blockquoted text
>
> which ends here

Code
----

For **inline code**, enclose in backticks, e.g.

    I'm using `markdown`

generates: "I'm using `markdown`"

For **code blocks**, indent by four spaces or a single tab.

Links
-----

### Inline links ####

Examples:

    This is an [example link](http://example.com)
    This is an [example link](http://example.com "with a Title")

### Reference links ###

Example:

    There are differences between [standard markdown][1] and [gtf][2]
    
    [1]: http://daringfireball.net/projects/markdown/syntax "Markdown syntax"
    [2]: http://github.github.com/github-flavored-markdown/ "Github Markdown"

The references can contain letters and spaces, e.g. `[ny times]` instead of
`[1]` etc.

### Automatic links ###

Enclosing a URL in angle brackets e.g. `<http://example.com>` generates
<http://example.com>

Images
------

Similar to links, e.g.:

    ![alt text](/path/to/image.jpg "Title")

Appendix 1: Escape sequences for special characters
===================================================

Add a preceding \\ to escape the following characters (e.g. `\*` to get \*):

    \   backslash
    `   backtick
    *   asterisk
    _   underscore
    {}  curly braces
    []  square brackets
    ()  parentheses
    #   hash mark
    +   plus sign
    -   minus sign (hyphen)
    .   dot
    !   exclamation mark

Appendix 2: Generating HTML from markdown
=========================================

Various markdown implementations exist:

*   E.g. on Fedora 15 you can install a Python implementation via `yum`:

    `yum install python-markdown`

*   You can also obtain a perl implementation from Daring Fireball:

    [markdown_1.0.1.zip](http://daringfireball.net/projects/downloads/Markdown_1.0.1.zip)

The basic usage is:

    markdown INPUTFILE -f OUTPUTFILE

(without `-f` the generated HTML goes to `stdout`).
