# The Bullang Book

Bullang is a language you write once and transpile to six target languages. It
does not run your code — it generates code that a real toolchain compiles and
runs.

| Tool | Role |
|---|---|
| `bullang` | The language: grammar, parser, formatter, and the builtin catalogue |
| `bullarchy` | The toolchain: scaffold, transpile, format and check projects |
| `bullscript` | A small pipe-only interpreted language — REPL and `.busc` scripts |

`bullang` and `bullarchy` live in one repository. `bullscript` is its own
language with its own grammar and its own interpreter, and shares no code with
Bullang; it is described in Part 2.

---

## Installation

```bash
cargo install --git https://github.com/The-Bullang-Foundation/Bullang.git bullang bullarchy
cargo install --git https://github.com/The-Bullang-Foundation/Bullscript.git
```

Reinstall over an existing version by adding `--force`. `bullarchy update` does
the same thing for you and rebuilds the GUI, which needs a Go toolchain.

---

# Part 1 — The Language

## The shape of the thing

Two ideas explain almost every rule in Bullang.

**It is read line by line.** One operation per bullet, no nested calls, no
precedence to trace. `a + b + c` is not an expression; it is two bullets.

**Everything must translate faithfully to all six backends.** A feature earns
its place by having an honest counterpart in Rust, Python, C, C++, Go *and*
Java. Features that could not — references, closures, function types, fixed
arrays — were removed rather than approximated in five languages and faked in
the sixth.

---

## Types

| Type | Meaning |
|---|---|
| `i64` | 64-bit integer |
| `f64` | 64-bit float |
| `bool` | `true` / `false` |
| `String` | UTF-8 string |
| `Tuple[A, B]` | A pair. Written `Tuple[A, B]`, not `(A, B)` |
| `()` | The unit type — no return value |

Struct and enum types are declared in `inventory.bu` and usable by name in the
same folder and one rank above.

There is no collection type. `Vec[T]` and `[T; N]` were removed: they could be
written in a signature but never constructed, so a function could declare a
return value no program could produce. Collections will come back as one
designed feature — literals, indexing, length and iteration together — or not
at all.

`()` is the only spelling of unit. `Unit` is gone.

---

## Functions

```
let calculus(a: i64, b: i64, c: i64) -> result: i64 {
    (a, b) : a + b -> {ab};
    (c, ab) : ab * c -> {result};
}
```

A function is `let`, a name, a parameter list, an optional output declaration,
and a body of bullets. The output declaration names the value *and* its type:
`-> result: i64`. Omit it entirely and the function returns nothing.

The last bullet must bind the declared output, by that name. A function with no
declared output must end in a discarding bullet:

```
let announce() {
    (1, "hello\n") : builtin::out -> {};
}
```

`-> {}` throws the value away. `builtin::out` returns a byte count, and this
function returns nothing, so the count is discarded — that is what `{}` says.

---

## Pipes

A bullet is:

```
(inputs) : expression -> {binding};
```

The inputs are values the bullet reads. What the expression *does* with them
depends on what it is, and there are exactly four cases:

| Bullet | Means |
|---|---|
| `(a, b) : a + b -> {sum};` | evaluate `a + b` |
| `(a, b) : add -> {sum};` | call `add(a, b)` |
| `(s) : builtin::to_upper -> {r};` | call the builtin with `s` |
| `(x) : some_fn(x, 2) -> {r};` | call `some_fn(x, 2)` as written |

A bare name with inputs is a call and the inputs are its arguments. Anything
that already stands on its own — an operation, a literal, a call with its own
argument list — is used as-is, and the inputs are simply the values it reads.

A binding is assigned once. Reusing a name is an error, which is what keeps a
bullet chain readable top to bottom.

---

## Expressions

One operation per bullet:

```
(a, b) : a + b -> {sum};
(s) : s[0] -> {first};
(s) : s[1..4] -> {slice};
(p) : p.x -> {x};
(b) : !b -> {flipped};
```

Operators: `+ - * / %`, `== != < > <= >=`, `&& ||`, unary `!` and `-`.

String templates interpolate bindings by name:

```
(name) : "Hello, {name}!" -> {greeting};
```

There is no `?`. Error propagation was removed along with the types it
propagated.

---

## Calling other functions

```
(a, b) : add -> {sum};
(a, b) : add(a, b) -> {sum};
```

Both forms work; the first is idiomatic. You may call any function listed in a
child folder's inventory. Skirmish files cannot call other functions at all —
they are leaves.

Function *references* (`&f`) and closures were removed: there is no honest
counterpart in all six backends, and passing a function around is not something
a line-by-line reading can follow.

---

## Builtins

Written `builtin::name`. The core set is deliberately small — anything beyond it
is a package installed with `bullarchy add` and declared with `#use:`.

Run `bullarchy stdlib` for the current list.

### Math
| Signature | |
|---|---|
| `min(a: i64, b: i64) -> i64` | smaller of two integers |
| `max(a: i64, b: i64) -> i64` | larger of two integers |

`min` and `max` take two integers. They previously claimed to take an array.

### Conditions
| Signature | |
|---|---|
| `tern(cond: bool, a: T, b: T) -> T` | `a` if `cond`, else `b` |
| `swap(a: T, b: T) -> Tuple[T, T]` | the same two values, reversed |

`tern` takes a condition. The old four-argument form folded a comparison into
the call, so you had to know the convention to see what was being tested.

### String
| Signature | |
|---|---|
| `to_upper(s: String) -> String` | uppercase |
| `to_lower(s: String) -> String` | lowercase |
| `trim(s: String) -> String` | strip leading and trailing whitespace |
| `starts_with(s: String, p: String) -> bool` | prefix test |
| `ends_with(s: String, p: String) -> bool` | suffix test |
| `replace_str(s, from, to: String) -> String` | replace every occurrence |
| `i64_to_str(x: i64) -> String` | integer to string |
| `str_to_i64(s: String) -> i64` | string to integer, **0 if it does not parse** |
| `len(s: String) -> i64` | length in **characters** |

`len` counts characters on every backend, and works on strings only. It used to
count bytes on three backends and characters on two, so a non-ASCII string gave
different answers depending on the target.

`str_to_i64` returns 0 on failure on every backend. It used to throw on Python,
C++ and Java, so the same program aborted on three targets and continued on the
other three.

`i64_to_str` and `str_to_i64` were called `to_string` and `parse_i64`.

### I/O
| Signature | |
|---|---|
| `in(fd: i64) -> String` | read one line; empty string at end of file |
| `out(fd: i64, content: String) -> i64` | write; returns bytes written |
| `open(path: String, mode: String) -> i64` | open in mode `r`, `w`, `a` or `rw` |
| `close(fd: i64)` | close a descriptor |
| `time() -> i64` | Unix timestamp in seconds |

**A file descriptor is not an operating-system descriptor.** It is an index into
a table the generated program keeps for itself: 0, 1 and 2 are stdin, stdout and
stderr, and `open` allocates from 3 up. That indirection is what lets the same
program work on Windows, where native handles are not integers. Closing 0, 1 or
2 does nothing.

### System
| Signature | |
|---|---|
| `argc() -> i64` | number of command-line arguments |
| `args(i: i64) -> String` | argument at index `i`; index 0 is the program |
| `exit(code: i64)` | exit with a status |
| `env(key: String) -> String` | environment variable, empty if unset |
| `sleep(ms: i64)` | sleep for milliseconds |
| `run(cmd: String) -> i64` | run a shell command, returns its exit code |

`args` is indexed rather than returning a list, because Bullang has no
collection type — the old `args() -> [String]` declared a value no program could
hold.

---

## Native escape blocks

An escape block is a macro: opaque, copied byte for byte, never parsed and never
reindented.

```
let fast_path(n: i64) -> r: i64 {
@rust
    let r = n.wrapping_mul(2);
    r
@end
}
```

Rules:

- **One block per function.** A function is either Bullang or an escape block.
- The backend name must be followed by a newline.
- The block's backend must match the folder's effective `#lang`.
- Bullang reads nothing inside it. Whatever it needs, it must be declared with
  `#lib:` — nothing is inferred from the block's contents.

Backends: `@rust`, `@python`, `@c`, `@cpp`, `@go`, `@java`. A `@c` block is
accepted in a C++ folder.

---

## Structs

Declared in `inventory.bu`, not in source files:

```
struct Point { x: i64, y: i64 }
```

Field access is a bullet like any other: `(p) : p.x -> {x};`

A type name must be unique across the whole project. Two folders declaring
different `Point`s is an error, because generated code puts every type in one
namespace.

---

## Enums

```
enum Direction { North, South, East, West }
```

Used as `Direction.North`.

---

## Generics

```
let pick[T](a: T, b: T) -> r: T {
    (a, b) : a > b -> {r};
}
```

Bounds are derived from what the body does: a comparison needs ordering,
equality needs equality, and a value merely passed through needs neither.

---

# Part 2 — BullScript

BullScript is a separate language. It shares no code, no grammar and no file
extension with Bullang: `.busc`, not `.bu`. It is interpreted, and it exists for
the small things — a pipe at a prompt, a script you keep around.

## The language

Four types: `i64`, `f64`, `bool`, `String`. One pipe form:

```
(inputs) : expression-or-call -> {binding: type};
```

Bindings carry their type, because there is no separate declaration to carry it.

```
(a: i64, b: i64) : a + b -> {sum: i64};
(1: i64, "hello\n": String) : builtin::out -> {ok: bool};
```

Every pipe is type checked before anything runs, so a type error can never
surface after a `builtin::out` has already printed.

## Builtins

| Signature | |
|---|---|
| `add(a: i64, b: i64) -> i64` | sum; errors on overflow |
| `to_upper(s: String) -> String` | uppercase |
| `to_lower(s: String) -> String` | lowercase |
| `trim(s: String) -> String` | strip surrounding whitespace |
| `i64_to_str(x: i64) -> String` | integer to string |
| `str_to_i64(s: String) -> i64` | string to integer, **0 if it does not parse** |
| `out(fd: i64, content: String) -> bool` | write; no newline appended |
| `in(fd: i64) -> String` | read one line, newline stripped |
| `open(path: String, mode: String) -> i64` | open in mode `r`, `w`, `a` or `rw` |
| `close(fd: i64) -> bool` | close a descriptor |
| `run(cmd: String) -> bool` | run a shell command |
| `capture(cmd: String) -> String` | run a command, return its output |

This is a much smaller set than Bullang's, and not the same one. Where a name
appears in both, it means the same thing — `str_to_i64` returns 0 on a string
that does not parse here exactly as it does there, because two languages
sharing a name while disagreeing on what it means would be worse than either
choice alone. `str_to_i64` ignores surrounding whitespace, so a line read with
`builtin::in` converts without a `trim` first.

## `.busc` scripts and their parameters

```bash
bullscript hello.busc
bullscript greet.busc "world"
```

**The first pipe's named slots are the script's parameters.** A literal slot is
a value the script already carries, so it keeps it:

```
(1: i64, "Hello, world!\n": String) : builtin::out -> {ok: bool};
```

runs with no arguments — everything it needs is written into it. Whereas:

```
(1: i64, who: String) : builtin::out -> {ok: bool};
```

takes one argument, for `who`. A literal is never supplied from the command line
and never overridden by it.

That rule is different for bag entries, and deliberately so — see below.

## The bag

The bag is a registry of named scripts, kept in `~/.bullscript`.

| Directive | |
|---|---|
| `bag::add <path> <name>` | parse, check and store a `.busc` file under `<name>` |
| `bag::remove <name>` | remove one entry |
| `bag::list` | list your entries |
| `bag::export <path>` | write every entry into one `.zip` |
| `bag::import <path>` | read every `.busc` in a `.zip` into your bag |

`bag::add` copies the script in, so the bag owns its copy: moving or deleting
the original cannot break an entry, and editing the original does not update it.

Call an entry from a pipe:

```
(4: i64) : bag::double -> {r: i64};
```

**Here every slot is filled by the caller, literals included.** That is what
makes an entry reusable: an entry written

```
(1: i64, who: String) : builtin::out -> {ok: bool};
```

can be sent to stdout or stderr by whoever calls it, because the `1` is a slot
rather than a fixed value. On the command line there is no caller, which is why
the rule differs there.

### Sharing a bag

```
bag::export ~/mybag.zip      # hand the file to someone
bag::import ~/mybag.zip      # they read it into their own bag
```

The archive is an ordinary zip that any tool can open; each entry is named after
its bag entry, so the round trip preserves names. An entry that already exists
is replaced.

**An imported archive is trusted.** Its scripts are copied in as they are, with
no parse or type check — vouching for an archive's coherence is yours to do,
exactly as it is for any file you point `bag::add` at. An incoherent entry fails
when you call it.

Two things still hold, and neither is a trust check: only the *file name* of
each archive entry is used, never the path recorded in it; and an entry whose
name is not an identifier is skipped, because it could never be called as
`bag::<name>`.

## Recording

`record::start` captures every pipe you type until `record::end`, which offers
to save the result to the bag. Only pipes that parsed, checked and ran are kept.

---

# Part 3 — Bullarchy

```bash
bullarchy              # the GUI
bullarchy --cli        # the REPL
bullarchy <command>    # one command and exit
bullarchy --help       # every command; <command> --help explains one
```

## Project structure

A project is a folder hierarchy, and every folder has a **rank**:

```
war → theater → battle → strategy → tactic → skirmish
```

| Depth | Hierarchy |
|---|---|
| 1 | skirmish |
| 2 | tactic → skirmish |
| 3 | strategy → tactic → skirmish |
| 4 | battle → strategy → tactic → skirmish |
| 5 | theater → battle → strategy → tactic → skirmish |
| 6 | war → theater → battle → strategy → tactic → skirmish |

Every folder holds an `inventory.bu` and its `.bu` source files. A function
declared at a lower rank is callable one rank above.

### inventory.bu

```
#rank: skirmish;
#lang: c;
#lib: stdio.h;
#use: mathlib;

struct Point { x: i64, y: i64 }

parser : parse_line, parse_word;
```

| Directive | |
|---|---|
| `#rank:` | this folder's rank — required, and first |
| `#lang:` | target backend for this subtree — optional |
| `#lib:` | a native header or import **of the target language** |
| `#use:` | a **Bullang package** installed with `bullarchy add` |

`#lib` and `#use` are different things and are spelled differently. `#lib:
stdio.h;` is a C include; `#lib: os/exec;` is a Go import. `#use: mathlib;` is a
Bullang package whose builtins then become available.

Nothing is inferred from your escape blocks. If a `@go` block calls
`strings.ToUpper`, say `#lib: strings;`.

### The limits

These are not arbitrary; they are what keeps a project readable at a glance.

| Limit | |
|---|---|
| 5 | sub-folders per folder |
| 5 | source files per folder |
| 5 | functions per file |
| 5 | functions in `main.bu` |
| 5 | bullets per function |
| once | a binding may be assigned |

### Language regions

A subtree may declare its own `#lang`. That subtree is a **region**: the
effective language is the nearest ancestor's that declares one.

Each region is transpiled independently, to its own language, into its own
output directory, with its own build file.

**A call may not cross a region boundary.** Bullang generates no FFI, so there
would be nothing to generate the call into. Move one of the two functions, or
remove the `#lang` so both are in one region.

`main.bu` belongs at the root of its region, and nowhere else.

---

## init

```bash
bullarchy init my_project
bullarchy init my_project --depth 3
bullarchy init my_project --lang c --lib stdio.h
bullarchy init my_project --blueprint blueprint.bu
```

| Option | |
|---|---|
| `--depth N` | hierarchy depth 1–6, default 2. Ignored in blueprint mode |
| `--lang ext` | `rs` `py` `c` `cpp` `go` `java` — the extension, not the name |
| `--lib header` | a `#lib` entry, repeatable |
| `--blueprint file` | build the tree from a blueprint |
| `--path dir` | where to create it |

`init` writes a `main.bu` you can run immediately, so a new project passes
`check` and transpiles without editing anything.

The project name must be an identifier. `--lang` must be an extension: `rs`, not
`rust`. `--lib` is not validated — its spelling belongs to the target language.

---

## convert

```bash
bullarchy convert my_project
bullarchy convert my_project -e py
bullarchy convert path/to/file.bu
bullarchy convert path/to/file.bu -e go -o out.go
```

| Option | |
|---|---|
| `-e, --lang ext` | language override; without it, the project's `#lang` decides |
| `-o, --out file` | output file — single-file mode only |

Output is written *beside* the project, as `_<name>`. A multi-region project
writes one directory per region.

`convert` runs the same validation and type checking as `check`, so it never
generates code for a project that would not have passed.

### Single-file convert

Converting one `.bu` file is the deliberate escape hatch: **language rules
apply, project rules do not.** No inventory, no ranks, no limits. Unknown type
names pass through unchecked. The target comes from `-e`, defaulting to Rust.

### Backends

| Extension | Language | Notes |
|---|---|---|
| `rs` | Rust | |
| `py` | Python | |
| `c` | C | see *Strings in C* below |
| `cpp` | C++ | |
| `go` | Go | needs Go 1.21 (`min`, `max`, `slices`) |
| `java` | Java | |

C, C++, Go and Java have no module system mirroring Bullang's folders, so their
output is flat. Two source files that would generate the same file name are an
error naming both.

### Strings in C

C has no owning string type, so a function returning `String` cannot return one
— the buffer would have to live somewhere, and every option except the caller's
frame is a leak or a dangling pointer.

So a Bullang function returning `String` becomes a C function returning `void`
that writes into a destination supplied first:

```
let loud(s: String) -> big: String { ... }
```
```c
void loud(char *big, char* s);
```

The caller declares the buffer immediately before the call. Nothing allocates.

A builtin's output size can be computed from its inputs, so its destination is
sized exactly:

```c
char t[ft_strlen(s) + 1];
ft_trim(t, s);
```

A *user* function's output length cannot be known in advance — the body could do
anything — so a caller-side buffer is a fixed `BU_STR_MAX` (4096 bytes). That
ceiling is the one real cost of the convention.

---

## fmt

```bash
bullarchy fmt
bullarchy fmt src/parser
bullarchy fmt --dry-run
```

Formats from the given folder down, or the whole project if no folder is given.

---

## check

```bash
bullarchy check
```

Structural validation, then type checking. Reports every error it can find, in a
stable order.

---

## Other commands

| Command | |
|---|---|
| `add` | list available packages |
| `add <name>` | install a package from the registry |
| `add <https://...>` | install from a git URL |
| `remove <name>` | uninstall a package |
| `stdlib` | list the core standard library |
| `editor-setup` | write LSP config for Vim, Neovim, Helix and Emacs |
| `update` | reinstall from the repository — the GUI needs Go |
| `lsp` | run the language server on stdin/stdout |

---

## Editors

```bash
bullarchy editor-setup
```

Writes LSP configuration for Vim, Neovim, Helix and Emacs, for **both**
languages: `.bu` is served by `bullarchy lsp`, `.busc` by `bullscript lsp`.
BullScript is configured only if it is installed.

Zed is the exception. It will not recognise a new language without an
extension, so there is no file to write — install `zed-bullang`, and
`zed-bullscript` for `.busc`, as dev extensions instead.

VS Code has an extension for each language too.

**BullScript's server** reports syntax and type errors as you type — a missing
semicolon, an unknown builtin, a binding whose declared type does not match
what the pipe produces. It runs the same lexer, parser and checker the
interpreter runs, so the underline in the editor and the error at the prompt
can never disagree.

---

# Part 4 — Quick Reference

## Function

```
let name(p: Type, q: Type) -> out: Type {
    (p, q) : p + q -> {out};
}

let no_return(p: Type) {
    (1, "text\n") : builtin::out -> {};
}
```

## Bullet

```
(inputs) : expression -> {binding};
(inputs) : function -> {binding};       function(inputs)
(inputs) : builtin::name -> {binding};
(inputs) : function(a, b) -> {binding}; as written
(inputs) : anything -> {};              discard
```

## BullScript pipe

```
(a: i64, b: i64) : a + b -> {sum: i64};
(4: i64) : bag::double -> {r: i64};
```

## Removed, and why

| Gone | Because |
|---|---|
| `Vec[T]`, `[T; N]` | a type with no values — returns as one designed feature |
| `&f`, closures, `Fn[...]` | no honest counterpart in six backends |
| `?` | removed with the types it propagated |
| `Unit` | one spelling: `()` |
| `(A, B)` as tuple syntax | `Tuple[A, B]` |
| nested calls, `a + b + c` | one operation per bullet |
| the interpreter | Bullang generates code; it does not run it |
