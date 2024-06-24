# Functional Programming Strategies in Scala with Cats

Copyright [Noel Welsh](http://twitter.com/noelwelsh) 2012-2024.

Artwork by [Jenny Clements](http://patreon.com/miasandelle).

Published by [Inner Product](https://inner-product.com/).

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.

Portions of Functional Programming Strategies in Scala with Cats are based on Scala with Cats by Dave Pereira-Gurnell and Noel Welsh, which is licensed under CC BY-SA 4.0.


## Overview

[Functional Programming Strategies][website] teaches the core concepts and techniques for building practical software in a functional programming style.


## Building

The build requires a lot of heavy machinery:
texlive, node, java, scala, and pandoc and pandoc-crossref.
There are all installed as part of the 
[CI job that builds the book](https://github.com/scalawithcats/scala-with-cats/blob/develop/.github/workflows/publish.yml).
This is always kept up to date, so refer to it for dependencies and their installation.

Once you have the dependencies installed, run `sbt` to open an SBT prompt.
From within `sbt` you can issue the following commands:

- `pdf` builds a PDF version in `dist/scala-with-cats.pdf`;
- `html` builds an HTML version in `dist/scala-with-cats.html`;
- `epub` builds an ePub version in `dist/scala-with-cats.epub`;
- `all` builds all three versions.

The `pdf`, `html`, and `epub` commands are each made of three smaller commands:

- `{foo}Setup` creates temp directories and builds JS/CSS prerequisites for the HTML/ePub versions;
- `mdoc` runs the files in `src/pages` through [mdoc](https://scalameta.org/mdoc/) to compile and run the Scala snippets;
- `{foo}Pandoc` runs the output of `mdoc` through [pandoc](https://pandoc.org/) to produce the output.

There are also `tex` and `json` commands
(and commands like `texSetup`, `jsonPandoc`, and so on)
that build a LaTeX version of the book and a Pandoc AST respectively.
These are useful for debugging the build.


## Contributing

If you spot a typo or mistake,
please feel free to fork the repo and submit a Pull Request.
Add yourself to [contributors.md](src/pages/preface/contributors.md)
to ensure we credit you for your contribution.

If you don't have time to submit a PR
or you'd like to suggest a larger change
to the content or structure of the book,
please raise an issue instead.

[website]: https://scalawithcats.com/
