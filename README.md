# SystemVerilog Style Guide

created: July 11, 2024
updated: July 11, 2024
contributors:
  * @jonmayer

## Table Of Contents

TODO.

## Overview

This document is a style guide for authoring Verilog and SystemVerilog RTL.
The recommendations in this style guide are design to:

    * improve code readability and consistency.

    * promote best practices (or, at least, warn against known pitfalls).

<details>
  <summary>Why write a style guide?</summary>

  Once upon a time, I helped write the SystemVerilog style guide at Google.
  That is to say: I took a set of existing adhoc guides and merged them
  together, resolved their various conflicts, social engineered widespread
  adoption, and set up a committee for continuing to grow and refine the style
  guide over the next few years.

  One of my great regrets is that we never open-sourced the style guide before
  I left Google. However, a version of the style guide did leak out, as the
  Low-RISC style guide.  Unfortunately, the Low-RISC team made a bunch of
  changes to make the guide more compatible with their pre-existing coding
  style, and removed attribution for the original authors.

  I guess if they can steal the style guide, I can steal it back.  But heck,
  here's an opportunity to make the style guide even better by redrafting it
  from whole cloth, streamlining it to remove low-value recommendations, and
  organizing it in a clearer way.  Maybe I can even glue in some new things
  I've learned since I left Google to work at startups.  Okay lets gooooo!
</details>


## Basics

### Design Unit Files

Each module, package, or interface should have its own source file, and only
one module, package, or interface should be defined in each source file.

The base name of this file must be the same as the name of the module, package,
or interface defined therein.

File extensions shall be as follows:

    * `.sv` indicates a SystemVerilog design unit (module, package, or interface).
    * `.svh` indicates a SystemVerilog code snippet, suitable for including with
      the <code>`include</code> directive.  This is usually used for macros.

<details><summary>Why</summary>

  Consistent names make the source code easier to navigate.  The "one design
  element per source file" rule makes life easier for design automation
  scripts such as stitching tools or automated build dependency management.
</details>
<details><summary>Examples.</summary>

  * A SystemVerilog file defining a module named `foo` should be named `foo.sv`.
  * A SystemVerilog file defining a package named `bar` should be named `bar.sv`.
  * A SystemVerilog file defining an interface named `bum_if` should be named `bum_if.sv`.
  * A file containing a library of verilog preprocessor macros might be named `macros.svh`.
</details>

### Design Unit Names

No particular prefix or suffix is used for the naming of synthesizable modules
or packages.  However, all interfaces should be named with an `_if` suffix.
<details><summary>Why?</summary>

  A naming convention is needed to differentiate a module from an interface,
  because otherwise it is impossible to differentiate an instantiation of a module
  from an instantiation of an interface in context.
</details>
<details><summary>Examples.</summary>
  * `foo` is a valid name for a module.
  * `bar` is a valid name for a package.
  * `bum_if` is a valid name for an interface.
</details>

### Basic File Formatting

* Non-ASCII characters are forbidden.  <details><summary>Why?</summary>7-bit ASCII
  represents the lowest common denominator for support amongst a broad variety of
  editors and tools (simulation, lint, synthesis, etc.).</details>

* Indentation uses spaces, not tabs.  Indent two spaces for nesting, four spaces
  for line continuation. <details><summary>Why?</summary>

    By configuring everyone's editor to perform basic indentation the same way, we
    avoid needless churn when a file passes back and forth between two authors.
  </details>
