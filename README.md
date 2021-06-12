# LaTeX Pre-rendering for Pelican

`pelican-mathml` integrates LaTeX rendering directly into the pelican generation
process and eliminates the delay in displaying math you usually experience on
the web. It does so by hooking itself into docutils' reStructuredText parser as
well as the markdown package and processing the formulas with a LaTeX to MathML
or HTML translator, either
[latex2mathml](https://github.com/roniemartinez/latex2mathml) or
[KaTeX](https://github.com/KaTeX/KaTeX).
The generated HTML pages only contain the finished HTML/MathML output.
Therefore, you do not need to ship a KaTeX, MathJax or similar
javascript implementation with your website anymore and improve the
accessibility as well as the load time of your internet presence.

Note, that if using the KaTeX backend, you still need to include the KaTeX
stylesheets with your website.

For a demo (using the KaTeX backend) visit this [blog
post](https://martenlienen.com/blog/sampling-k-partite-graph-edges/). Notice how all
the formulas are just there. There is no loading and the website does not even
serve the javascript part of KaTeX.

Forked from [pelican-katex](https://github.com/martenlienen/pelican-katex).

## Installation

First of all, you need to have a Python3 installation.
The dependencies vary depending on the backend you wnat to use.
 - For latex2mathml you only need `latex2mathml`, eg via
   `pip3 install latex2mathml`.
 - For KaTeX, you need a `nodejs` installation to run `katex.js`.

Currently this module is not available via pip and
still uses the `pelican_katex` name.
To make it available in developement mode, clone this repo and run the
following inside the repo root:
```sh
pip3 install --user -e "$PWD"
```
Finally, add `"pelican_katex"` to the `PLUGINS` setting
in your pelican configuration file.

If you previously used `katex.js` in your template,
you can now remove it.

## Backends

The default backend is `latex2mathml`.

 - `latex2mathml` is faster, does not require `nodejs`
    and also doesn't require a large-ish CSS file and KaTeX-fonts
    to be loaded.
    However it does not support preamble.

    While not required, you might still want to add a few CSS lines however:
    ```css
    math {
        font-family: 'Latin Modern Math',math;
    }
    math[display='inline'] {
        font-size: 110%;
    }
    math[display='block'] {
        font-size: 135%;
        margin-block: 0.4em;
    }
    ```

 - **KaTeX** supports a preamble and can generate fallback pure-HTML
    for browsers with subpar MathML support.
    However it requires nodejs, is slower in generation and also requires a
    special CSS-file and fonts. You must therefore add eg this to your template:
    ```html
    <link rel="stylesheet"
      href="https://cdn.jsdelivr.net/npm/katex/dist/katex.min.css"
      crossorigin="anonymous">
    ```

    Note that if you only want to use KaTeX anyway it might be better to use
    [pelican-katex](https://github.com/martenlienen/pelican-katex) instead
    as I can't give much support for the KaTeX backend.

## Syntax

```
reStructuredText
~~~~~~~~~~~~~~~~

In rst you write inline math with the usual math role (:math:`f(x)`) or
block math with

.. math::

    \int \textrm{math block}.

# markdown

In markdown you get inline math in between $ signs, like $f(x) = \sqrt{x}$.
Note, that $ only creates a math environment if it is preceded by whitespace
or at the beginning of a block and followed by some non-whitespace character.
This is necessary so that you can still write about the 5$ in your pocket.
Block math is triggered with

$$\int \textrm{math block}.$$

Math blocks can have linebreaks but no empty lines.
```

## Configuration

The plugin offers several configuration options that you can set in your
`pelicanconf.py`.

```python
# nodejs binary path or command to run KaTeX with.
# KATEX_NODEJS_BINARY = "node"

# Path to the katex file to use. This project comes with version `0.10` of
# katex but if you want to use a different one you can overwrite the path
# here. To use a katex npm installation, set this to `"katex"`.
# KATEX_PATH = "/path/to/katex.js"

# By default, this plugin will redefine reStructuredText's `math` role and
# directive. However, if you prefer to have leave the docutil's defaults
# alone, you can use this to define a `katex` role for example.
# KATEX_DIRECTIVE = "katex"

# How long to wait for the initial startup of the rendering server. You can
# increasing it but if startup takes longer than one second, something is
# probably seriously broken.
# KATEX_STARTUP_TIMEOUT = 1.0

# Time budget in seconds per call to the rendering engine. 1 second should
# be plenty since most renderings take less than 50ms.
# KATEX_RENDER_TIMEOUT = 1.0

# Define a preamble of LaTeX commands that will be prepended to any rendered
# LaTeX code.
# KATEX_PREAMBLE = None

# Here you can pass a dictionary of default options that you want to run
# KaTeX with. All possible options are listed on KaTeX's options page,
# https://katex.org/docs/options.html.
# KATEX = {
#     # Abort the build instead of coloring broken math in red
#     "throwOnError": True
# }

# If set to True (the default) latex2mathml will be used,
# if set to False KaTeX will be used instead
# KATEX_MATHML = True
```

## Preamble (KaTeX only)

The `KATEX_PREAMBLE` option allows you to share definitions between all of your
math blocks across all files. It takes a string of any LaTeX commands you would
like, for example

```python
KATEX_PREAMBLE = r"""
\def\ceil#1{\lceil #1 \rceil}
\def\floor#1{\lfloor #1 \rfloor}
"""
```

If you have a large preamble, it might be nice to extract it into a `.tex` file.
Note, that pelican will not be aware of changes made to that file in autoreload
mode and you will have to restart pelican manually.

```python
from pathlib import Path
KATEX_PREAMBLE = Path("preamble.tex").read_text()
```

You can also add more definitions per file to the preamble with preamble-blocks
that do not produce any output.

```
reStructuredText
~~~~~~~~~~~~~~~~

.. math::
   :preamble:

   \def\pelican{\textrm{pelican}^2}

This definition will be available in subsequent blocks

.. math::

   \sqrt{\pelican}

or inline :math:`\pelican = 1`.

# markdown

In markdown it is not as easy to define properties of blocks, so we chose to
start a preamble block with an @ such as

$$@
\def\pelican{\textrm{pelican}^2}
$$

which works just the same in blocks

$$\sqrt{\pelican}$$

or inline $\pelican = 1$.
```
