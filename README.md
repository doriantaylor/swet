# SWeT: Standard Web Templates

Template processing is something nearly every website has to do, and it's frankly baffling that there is no standard way to do it. **Oh wait, there is.** It's called [XSLT](https://www.w3.org/Style/XSL/), and the consortium of Web browsers [has coordinated to eliminate it](https://github.com/whatwg/html/issues/11523).

**XSLT got many things right** that so-called "modern" client-side Web templating systems still fail to accomplish:

* it is fully [declarative](https://en.wikipedia.org/wiki/Declarative_programming) (some templating systems are, many aren't).
* it is purely-functional, meaning it can't produce side effects.
* it works by transforming DOM trees instead of stitching together tag soup.
* it is incapable of generating malformed output (unless you really go out of your way to force it to).
* it _only_ concerns manipulating markup (and later versions, DOM-like data structures in general) and nothing else.
* it operates out of band from JavaScript, meaning you can still manipulate the presentation while guaranteeing that the page itself only exhibits deterministic behaviour.

However, XSLT is not without its shortcomings:

* it's old
* the functionality sucks
* it only operates over XML
* XPath is haaard 😢
* the syntax is truly awful

_Yes_, it's old, but it's only old in _Web browsers_. XSLT is actually still used extensively in publishing, and has been updated several times since it was first released in 1999. In fact, a [draft of version 4.0](https://qt4cg.org/specifications/xslt-40/Overview.html) shipped the same week as that GitHub issue moving to nuke XSLT from the Web was posted.

_Yes_, the functionality of XSLT 1.0 sucks. That was pretty much all fixed in [XSLT 2.0](https://www.w3.org/TR/xslt20/). In _2007_. Did the browsers update their implementations? No they did not. In fact the WebKit team blamed the maintainer of the (open-source) LibXSLT for not doing it. For free. (This does not excuse Mozilla, as they have their own implementation.)

_Yes_, it only operates over XML, and XML is out of style. (Actually, [XSLT 3.0 can process JSON](https://www.w3.org/TR/xslt-30/Overview.html#json).)

_Yes_, XPath is hard (I guess?? It's not much harder than CSS selectors). That point is conceded(ish).

_Yes_, the syntax absolutely is awful. It is a terrible experience to read and _especially_ to write. That point is absolutely conceded.

# Don't Throw It Away, Fix It!

I humbly submit that these are all tractable problems.

* Define rules for handling HTML DOM trees (easy: implied default namespace, normalize tags to lower case).
* Define an alternative hook for HTML analogous to the `xml-stylesheet` processing instruction (easy: `<link>` and `<script>` tags exist, so does the `Link:` HTTP header).
* Define a mapping for a subset of CSS selectors to XPath; make it possible to use either one (easy: pretty much anything you can say in a CSS selector — except pseudo-classes like `:hover` which would be meaningless — you can say in XPath).
* Define a new, compact, easy-to-type syntax, similar to [Sass](https://sass-lang.com/) or [RelaxNG](https://relaxng.org/compact-tutorial-20030326.html) (i.e., the fun part).

Then, it would simply be a matter of writing a little transpiler that transforms that syntax into the canonical XML one that nobody likes typing. There are [already](https://github.com/SupsoOrg/sup-xml) (at least) [three](https://github.com/Paligo/xee) [implementations](https://gitlab.gnome.org/World/Rust/markup-rs/xrust) of XSLT 3.0 written in Rust which could be compiled to WASM. They're still young but a little elbow grease and some tokens could bring one or both of them up to conformance. Then it would be a matter of lobbying the W3C to adopt the hook and the HTML semantics profile, which could be the same specification, and either the W3C or WHATWG to adopt the syntax specification (or heck, just put it out as an IETF RFC).

Then, just like JSON, which got its own DOM implementation because people kept `eval()`ing it, we yell at the browsers to put XSLT back. Then we have a nice, clean, sensible, *standard* Web templating system forevermore.

# An Example

Here is an idea for what SWeT could look like, which I apparently sketched out [all the way back in 2019](https://doriantaylor.com/file/xslt-mockup):

```
# xslt compact syntax mockup number one

# shell-style comments (of course)

# turtle/sparql style namespace declarations
prefix html: <http://www.w3.org/1999/xhtml>

# idea is we have an inventory of reserved words which if you need to
# you can override with % just like haml. other than that the first
# thing on the line is treated like an element name

# abridged template syntax? name/mode, match, priority
template#name html:foo 100 {
  # params only appear at the beginning of a template, maybe this?
  $$ { hurr: "durr" }

  # abridged command elements of course (this would be xsl:for-each)
  each html:bar {

    # do we do it like haml? i'm not a fan of semantic indentation,
    # but something like this might be okay:
    p all subsequent tokens become a text node; p next element on same line

    p 'quotes optional; eg for if you want to include a literal semicolon'

    # i'm much more partial to something like this
    p id: 'foo', class: $var {
      child element
    }

    # automatically determine between xsl:if and xsl:choose; xpath
    # functions pass through, of course
    if position() mod 2 {
      # this could be a way to apply templates with a mode
      apply#odd .
    }
    else {
      apply#even . param: 'val', other:param {
         something more elaborate
      }
    }
  }
}
```

# We Can Just Do Things

Love,

[Dorian](https://doriantaylor.com/)
