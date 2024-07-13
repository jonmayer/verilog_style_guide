# SystemVerilog Style Guide

created: July 11, 2024<br\>
updated: July 11, 2024<br\>
contributors:
  * @jonmayer

## Table Of Contents

TODO.

## Overview

This document is a style guide for authoring Verilog and SystemVerilog RTL.
The recommendations in this style guide are design to:

    * improve code readability and consistency.

    * promote best practices (or, at least, warn against known pitfalls).

This style guide also attempts to be minimalist.  Guidance that doesn't
improve anyone's life is omitted.

<details>
  <summary>Why?</summary>

  Once upon a time, I helped write the SystemVerilog style guide at Google.
  That is to say: I took a set of existing adhoc guides and merged them
  together, resolved their various conflicts, social engineered widespread
  adoption, and ran a committee for continuing to grow and refine the style
  guide.  Over the next few years, the style guide was honed by many hands
  into something useful that I was proud of.

  One of my great regrets is that we never open-sourced the style guide before
  I left Google. However, the style guide did eventually leak out, as the
  [Low-RISC style
  guide](https://github.com/lowRISC/style-guides/blob/master/VerilogCodingStyle.md).
  The Low-RISC team made a bunch of changes (making the guide compatible with
  their pre-existing coding style), and removed attribution for the original
  authors.

  I guess if they can steal the style guide, I can steal it back.  But heck,
  here's an opportunity to make the style guide even better by redrafting it
  from scratch, streamlining it to remove low-value recommendations, and
  organizing it in a clearer way.  Maybe I can even glue in some new things
  I've learned since I left Google.  Okay lets gooooo!
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

### Basic File Formatting

#### Use ASCII characters

Non-ASCII characters are forbidden.

<details><summary>Why?</summary>
  7-bit ASCII represents the lowest common denominator for support amongst a
  broad variety of editors and tools (simulation, lint, synthesis, etc.).  For
  better or worse, unicode support is still not universal.
</details>

#### Indentation

Indent using spaces, not tabs.

Indent two spaces for nesting (`begin`/`end` blocks, functions, tasks, etc).

Indent four spaces for line continuations.

<details><summary>Why?</summary>

  Indentation tends to be automated by modern editors. By configuring
  everyone's editor to perform indentation the same way, we avoid needless
  version control churn when a file passes back and forth between two different
  authors.
</details>
<details><summary>Examples.</summary>

```systemverilog {.good}
always @(posedge clk) begin : posedge_clk
  // Anything between a begin/end pair is indented by 2 spaces.
  if (le) q <= d;
end : posedge_clk
```

```systemverilog {.good}
assign foo =
    (state == STATE_IDLE) ||  // note four spaces for a continued line.
    (state == STATE_ERROR) ||
    (state == STATE_PAUSED);
```

</details>

#### Tabularize

Format code as a table in places where it improves readability, such as in variable
declarations or port lists.

<details><summary>Why?</summary>
  Aligned text makes it easier to spot similarities and differences between,
  for example, groups of variables that share a common prefix.  This makes it
  easier to spot typos, and makes the information presented easier to group
  and visualize.
</details>

<details><summary>Examples.</summary>

```systemverilog {.good}
module foo (
  input  logic            clk,
  input  logic            rst_n,

  input  logic            position_valid,
  output logic            position_ready,
  input  logic [15:0]     position_x,
  input  logic [15:0]     position_y,

  output logic            angle_valid,
  input  logic            angle_ready,
  output floating_point_t angle
);

foo_bar foo_bar (
  .clk,
  .rst_n,
  .px      (position_x),
  .py      (position_y),
  .angle   (angle)
);

endmodule : foo
```
</details>

#### Parenthesize

If you have to think for more than a moment about the correct order of
operators, please add parentheses to make your expressions unambiguous.

<details><summary>Examples.</summary>

```
// confusing and possibly incorrect:
assign foo = a == b ? c == d ? x : y : z;

// parens improve clarity:
assign foo = (a == b) ? ((c == d) ? x : y) : z;

// parens and indentation might even be best:
assign foo = (a == b)
    ? ((c == d)
        ? x
	: y)
    : z;
```
</details>

## Names

Naming things correctly is the second hardest problem in hardware design.
Clear, readable names are the bedrock on which clear, readable code is built.

### lower\_snake\_case and UPPER\_SNAKE\_CASE

Most names (modules, instance names, types, signals, etc.) are to be formatted as `lower_snake_case`.

The one exception is for symbols containing constants, such as parameters and
enumerated items, which are formatted as `UPPER_SNAKE_CASE`.

<details><summary>Why?</summary>

Being able to visually distinguish constants from variables improves the
readability of code.

We don't use lowerCamelCase or UpperCamelCase because, apocryphally, some ancient
physical design tools would convert all symbols to lowercase during synthesis,
making the symbols hard to read. Perhaps modern tools won't do this anymore,
but why take the chance?
</details>

### Use UPPER\_SNAKE\_CASE for certain things

### Module, Package, Interface Names

All interfaces should be named with an `_if` suffix.

No particular suffix is required for modules or parameters.

<details><summary>Why?</summary>

  Interface instantiation is syntactically indistinguishable from module
  instantation, so a naming convention is helpful to distinguish the two.
  
  Packages, on the other hand, are easily distinguished from modules by
  syntax and usage.
</details>
<details><summary>Examples.</summary>
  * `foo` is a valid name for a module.  ie. `foo u_foo(...)`.
  * `bar` is a valid name for a package, ie. `bar::my_type_t`.
  * `bum_if` is a valid name for an interface, ie. `bum_if u_bum_if(...)`.
</details>

### PD Boundary Modules

All physical design boundary modules (modules that will be place-and-routed
separately during hierarchical physical deisgn) should have names that end with `_top`.

<details><summary>Why?</summary>

PD boundary modules usually have a different set of requirements (ie.
registered inputs and outputs, limits on fanout, feedthrough insertion, etc.),
enforced by a different set of lint or synthesis design checks.  Clearly
labelling the PD boundaries in the module name makes this easier to automate.
</details>

### Instance Names

Instances should have the same name as the module or interface being
instantiated, but with a `u_` prefix.

If it is necessary to instantiate multiple instances of the smale module
in a source unit, a suffix should be appended to the name to distinguish
them.

<details><summary>Why?</summary>
This approach allows the undering module or interface type to be inferred
from the instance name, which is often useful when looking at waveforms
or synthesis reports.
</details>

<details><summary>Examples.</summary>
```
my_multiplier u_my_multiplier_foo (
    .clk,
    .rst_n,
    .op0     (foo),
    .op1     (foo),
    .product (foo_squared));

my_multiplier u_my_multiplier_bar (
    .clk,
    .rst_n,
    .op0     (bar),
    .op1     (bar),
    .product (bar_squared));

my_adder u_my_addr (
    .clk,
    .rst_n,
    .op0 (foo_squared),
    .op1 (bar_squared),
    .sum (foo_squared_plus_bar_squared));
```
</details>

### Signal Names

#### Clocks and Resets

If a module is fully synchronous (has only one clock), clock
should be named `clk` and reset should be named `rst_n` (for
a low-active reset) or `rst` (for a high-active reset).

<details><summary>Why?</summary>

Precedent.
</details>

If a module has multiple clocks, a prefix should be prepended
to every signal and port name to identify what clock domain
that signal is in.

<details><summary>Why?</summary>

Adding a prefix to every port and signal is onerous, but helps spot CDC
violations early, and encourages designers to segregate different clock domains
into individual modules.  This leaves multi-clock-domain design to
the assemblage modules, which can typically be generated using stitching
tools.
</details>

<details><summary>Examples.</summary>
```
module cdc_fifo (
    input  logic        cmp_clk,
    input  logic        cmp_rst_n,
    input  logic [31:0] cmp_data,
    input  logic        cmp_valid,
    output logic        cmp_ready,

    input  logic        rqr_clk,
    input  logic        rqr_rst_n,
    output logic [31:0] rqr_data,
    output logic        rqr_valid,
    input  logic        rqr_ready);
```

## Open Style Questions

There are a few style choices where there is room for debate and the best
practices solution is not obvious.  This section tracks these questions
and solicits input from the community.

### Data structures: `packed` or `unpacked`?

`packed` data structures are easier to cast to and from opaque bitvectors.
`unpacked` data structures enable strong type checking.  If only SystemVerilog
had chosen to support a `packed` option that still allowed compile time type
checking.

### Unions

Unions tend to create very strangely munged named at the PD stage of a project.
Their functionality can largely be implemented through casting.  Should their
use be discouraged?

## Final words

### Is that all there is?

The original style guide I worked on at Google was much longer than this. This
was largely due to the needs of a team that was writing a verilog code
formatting tool, and they expected the style committee to guide them on a
variety of trivial formatting decisions.  As a consequence, the guide grew to
include a lot of low-impact but decisive guidance (add one space here, omit one
space there) -- great for code formatting tools, not so great for humans.

This style guide says goodbye to all of that.

### Please help

Please don't be shy if you have a suggestion to make the style guide better.

