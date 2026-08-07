# The Bullang Book

Bullang is a language that lets you write code once and transpile it to multiple target languages. Three tools, three roles:

| Tool | Role |
|---|---|
| `bullang` | Language definition and stdlib reference |
| `bullscript` | A small pipe-only interpreted language — REPL and `.busc` scripts |
| `bullarchy` | Scaffold, transpile, format, and check projects |

---

## Installation

```bash
cargo install --git https://github.com/My-sidequests/Bullang.git
cargo install --git https://github.com/My-sidequests/Bullarchy.git
cargo install --git https://github.com/My-sidequests/Bullarchy-gui.git
cargo install --git https://github.com/My-sidequests/Bullscript.git
```

Reinstall over an existing version:
```bash
cargo install --git https://github.com/My-sidequests/Bullang.git    --force bullang
cargo install --git https://github.com/My-sidequests/Bullarchy.git  --force bullarchy
cargo install --git https://github.com/My-sidequests/Bullarchy.git  --force bullarchy-gui
cargo install --git https://github.com/My-sidequests/Bullscript.git --force bullscript
```

Update from within the tools:
```bash
bullang update
command -> update   # inside bullarchy
```

`bullscript` has no in-app update command — reinstall it with the `cargo install --force` command above.

---

# Part 1 — The Language

## Types

| Type | Description |
|---|---|
| `i32` | 32-bit integer |
| `i64` | 64-bit integer |
| `f64` | 64-bit float |
| `bool` | true / false |
| `String` | UTF-8 string |
| `(A, B)` | Tuple |
| `[T; N]` | Fixed-size array |
| `Unit` | No return value |

T stands for Type, meaning the user can choose a type for the array. N stands for the number of slots in the array. For exemple, [i32; 4] will create an array of 4 i32 units.

Struct and enum types are defined in `inventory.bu` and usable by name anywhere in the same folder and one rank above.

---

## Functions

Here's an example of what a function must look like.

```
let calculus(a: i32, b: i32, c: i32) -> result: i32 {
    (a, b) : a + b -> {ab};
    (c, ab) : ab * c -> {result};
}
```

Here's an example of a function without a returned value.

```
let name(param: Type) {
    ("Hello/n") : builtin::out -> {};
}
```

- All functions are always public.
- A function body holds **up to 5 pipes**, or **one escape block** — not both.
- The last pipe's binding is the return value. Look at the calculus function above to get it.

---

## Pipes

A pipe, also known as a bullet, is one transformation step inside a function.

```
(inputs) : expression -> {binding};
```

| Part | Description |
|---|---|
| `(inputs)` | Values passed into the expression. Idents or literals. |
| `: expression` | The transformation — arithmetic, a function call, a builtin. |
| `-> {binding}` | Name the output. Use `{}` to discard. |

```
(a, b) : a + b       -> {sum};
(sum)  : builtin::to_string -> {sum_str};
(sum_str) : "Result: {sum_str}\n" -> {msg};
(1, msg) : builtin::out -> {};
```

---

## Expressions

```
a + b     a - b     a * b     a / b     a % b
a == b    a != b    a < b     a <= b    a > b    a >= b
a && b    a || b    !a        -a
```

String interpolation:
```
"hello {name}, you are {age} years old"
```

String index and slice:
```
(s) : s[0]    -> {first_char};
(s) : s[1..4] -> {substr};
```

---

## Calling other functions

Pass inputs implicitly — the input list is forwarded as arguments:
```
(a, b) : add   -> {result};
(name) : shout -> {loud};
```

---

## Builtins

Use in a pipe with implicit inputs:
```
(s) : builtin::to_upper -> {result};
```

### Math

| Builtin | Signature | Description |
|---|---|---|
| `abs` | `abs(n) -> n` | Absolute value |
| `pow` | `pow(base, exp) -> i64` | Integer power |
| `powf` | `powf(base, exp) -> f64` | Float power |
| `sqrt` | `sqrt(n) -> f64` | Square root |
| `log` | `log(x, base) -> f64` | Logarithm |
| `exp` | `exp(n) -> f64` | e^n |
| `min` | `min(a, b) -> n` | Minimum |
| `max` | `max(a, b) -> n` | Maximum |
| `clamp` | `clamp(v, lo, hi) -> n` | Clamp value |

### Conditions

| Builtin | Signature | Description |
|---|---|---|
| `tern` | `tern(cond, a, b) -> a or b` | Ternary — returns a if true, b if false |

### String

| Builtin | Signature | Description |
|---|---|---|
| `to_upper` | `to_upper(s) -> String` | Uppercase |
| `to_lower` | `to_lower(s) -> String` | Lowercase |
| `trim` | `trim(s) -> String` | Strip whitespace |
| `len` | `len(s) -> i64` | Character count |
| `starts_with` | `starts_with(s, p) -> bool` | Prefix check |
| `ends_with` | `ends_with(s, p) -> bool` | Suffix check |
| `replace_str` | `replace_str(s, from, to) -> String` | Replace all occurrences |
| `to_string` | `to_string(x) -> String` | Any value to string |
| `parse_i64` | `parse_i64(s) -> i64` | String to integer |

### Algorithms

| Builtin | Signature | Description |
|---|---|---|
| `swap` | `swap(a, b) -> (b, a)` | Swap two values |
| `insertion_sort` | `insertion_sort(arr) -> Array` | Sort (insertion) |
| `quick_sort` | `quick_sort(arr) -> Array` | Sort (quicksort) |
| `merge_sort` | `merge_sort(arr) -> Array` | Sort (mergesort) |
| `radix_sort` | `radix_sort(arr) -> Array` | Sort (radix) |

### I/O

| Builtin | Signature | Description |
|---|---|---|
| `in` | `in(fd) -> String` | Read a line from fd |
| `out` | `out(fd, s) -> i64` | Write to fd, returns bytes written |
| `open` | `open(path, mode) -> i64` | Open file, returns fd. Modes: r w a rw |
| `close` | `close(fd)` | Close a file descriptor |
| `time` | `time() -> i64` | Unix timestamp in seconds |

File descriptors: `0` stdin · `1` stdout · `2` stderr · `3+` opened files.

### System

| Builtin | Signature | Description |
|---|---|---|
| `args` | `args() -> Array` | Command-line arguments |
| `exit` | `exit(code)` | Exit with code |
| `env` | `env(key) -> String` | Read environment variable |
| `sleep` | `sleep(ms)` | Sleep for N milliseconds |

---

## Native Escape Blocks

You can use an escape block, for whenever you need something specific in a target language

```
let add(a: i32, b: i32) -> result: i32 {
    @rust
        let result = a + b;
    @end
}
```

- One escape block per function.
- Cannot mix pipes and escape blocks in the same function.
- Supported backends: `@rust` `@python` `@c` `@cpp` `@go` `@java`

---

## Structs

Defined in `inventory.bu`:
```
struct Point {
    x : i32,
    y : i32,
}
```

Field access in a pipe:
```
(p) : p.x -> {x_coord};
```

---

## Enums

Defined in `inventory.bu`. C-style. 

```
enum Direction {
    North,
    South,
    East,
    West,
}
```

Usage:
```
(Direction.North) : some_function -> {result};
```

---

## Generics

```
let identity[T](x: T) -> result: T {
    (x) : x -> {result};
}
```

---

# Part 2 — Bullscript

BullScript is a small, pipe-only interpreted language of its own — not a
toolbox of commands over Bullang. It borrows Bullang's pipe syntax for
familiarity, but is its own grammar and evaluator; Bullang itself is never
modified or extended by BullScript. `bullscript` is the interpreter
itself — there is no command dispatcher.

```bash
bullscript
bullscript ->
```

Running `bullscript` with no arguments drops straight into that prompt.
This *is* the whole program.

```bash
bullscript path/to/script.busc
```

Runs a `.busc` file non-interactively (script mode) instead.

## The language

A BullScript program — a `.busc` file, or a line typed at the prompt — is
nothing but a sequence of pipes. There is no `let`, no functions, no
blocks, no escape blocks:

```
( <input>: <type>, ... ) : <callee-or-expr> -> { <name>: <type> } ;
```

- **Every input and every created binding always carries an explicit
  type** — no inference, unlike Bullang's own pipes.
- The middle section is either a call (`builtin::name` or `bag::name`,
  taking the pipe's inputs as its arguments, in order) or a bare
  expression over the pipe's own inputs (`+ - * /`, `== != < > <= >=`,
  `&& ||`, unary `-`/`!`, parens).
- `-> {}` discards the result; `-> {name: type}` creates or overwrites
  `name`.
- Exactly four types: `i64`, `f64`, `bool`, `String`. No tuples, no
  arrays.

```
(a: i64, b: i64) : builtin::add -> {result: i64};
(result: i64) : builtin::to_upper -> {out: String};
```

(The second pipe above is a type error — `to_upper` wants a `String`, not
an `i64`. Every step is checked.)

### `.busc` scripts and the bag

A `.busc` file *is* a sequence of pipes:

- The **first pipe's** typed inputs are the script's parameter list.
- The **last pipe's** binding is its return value.

`.busc` scripts are interpreted every run, not compiled — no build step,
no stored binary. The bag stores only `.busc` files; every callable
(builtin or bag entry) needs a declared prototype, so there's no path to
registering an arbitrary pre-built binary as a callable name.

```
bullscript -> bag::add double.busc double
bullscript -> (4: i64) : bag::double -> {r: i64};
```

### Directives

Typed bare at the prompt, not prefixed with anything:

| Directive | Action |
|---|---|
| `help` | Print the in-prompt help |
| `bag::add <path> <name>` | Register a `.busc` file under `<name>` (overwrites with a warning) |
| `bag::remove <name>` | Remove a single bag entry |
| `bag::list` | List your bag entries — builtins never appear here |
| `record::start` | Start capturing every pipe typed, verbatim |
| `record::end` | Stop, preview, and optionally save the recording as a new bag entry |
| `exit` | Quit. Ctrl+D also works; either discards an in-progress recording with a warning |

### Builtins

A small, fixed table — separate from Bullang's own stdlib in Part 1, and
not reused from it. Never stored in the bag, never removable.

| Builtin | Signature | Description |
|---|---|---|
| `builtin::add` | `(i64, i64) -> i64` | Addition |
| `builtin::to_upper` | `(String) -> String` | Uppercase |
| `builtin::to_lower` | `(String) -> String` | Lowercase |
| `builtin::trim` | `(String) -> String` | Strip whitespace |
| `builtin::out` | `(i64, String) -> bool` | Write to stream (`1` stdout, else stderr) |
| `builtin::run` | `(String) -> bool` | Run a shell command, return success/failure, discard output |
| `builtin::capture` | `(String) -> String` | Run a shell command, return stdout, no status info |

`run` and `capture` are split rather than combined: with no tuple type, a
single call can only bind one typed value, so status and output can't
come back from the same call.

---

# Part 3 — Bullarchy

```bash
bullarchy
command ->
```

## Project structure

A Bullang project is a folder hierarchy. Each folder has a **rank**.

```
war
└── theater
    └── battle
        └── strategy
            └── tactic
                └── skirmish
```

Each folder contains:
- `inventory.bu` — rank declaration, lang, lib, struct/enum definitions, filenames with functions
- One or more `.bu` source files

A function defined at a lower rank is available one rank above.

### Rank depths

| Depth | Hierarchy |
|---|---|
| 1 | skirmish |
| 2 | tactic → skirmish |
| 3 | strategy → tactic → skirmish |
| 4 | battle → strategy → tactic → skirmish |
| 5 | theater → battle → strategy → tactic → skirmish |
| 6 | war → theater → battle → strategy → tactic → skirmish |

### inventory.bu directives

| Directive | Description |
|---|---|
| `#rank: skirmish;` | Rank of this folder — required |
| `#lang: c;` | Target backend for this folder - optional |
| `#lib: stdio.h;` | External library - optional |

After directives: struct and enum definitions, then the file and functions:
```
filename : function1, function2, function3;
```

---

## Init

Scaffold a new project.

```
command -> init my_project
command -> init my_project --depth 3
command -> init my_project --lang c --lib stdio.h
command -> init my_project --blueprint blueprint.bu
```

| Option | Description |
|---|---|
| `--depth N` | Hierarchy depth 1–6. Default: 2 |
| `--lang ext` | Target language: rs py c cpp go |
| `--lib header` | Add a library directive (repeatable) |
| `--blueprint file` | Init from a blueprint.bu file |
| `--path dir` | Where to create the project |

---

## Convert

Transpile a project or a single file.
By default and if there's no escape blocks, convert transpile in Rust.

```
command -> convert my_project
command -> convert my_project -e py
command -> convert path/to/file.bu
command -> convert path/to/file.bu -o out.rs
```

| Option | Description |
|---|---|
| `-e ext` | Choose target language for folder conversion |
| `-n name` | Output folder name |
| `--out dir` | Output path (project mode) |
| `-o file` | Output file (single-file mode) |

### Backends

| Extension | Language |
|---|---|
| `rs` | Rust |
| `py` | Python |
| `c` | C |
| `cpp` | C++ |
| `go` | Go |
| `java` | Java |

---

## Fmt

Format all `.bu` files to canonical style.

```
command -> fmt
command -> fmt my_project
command -> fmt --dry-run
```

- Rewrites files in place.
- Escape block contents are never modified.
- `--dry-run` shows what would change without writing.

---

## Check

Validate and type-check from the current directory.

```
command -> check
```

Three passes in order:
1. Structure — rank hierarchy, inventory consistency, function declarations
2. Types — type checking across all bullets
3. Format — flags any file not in canonical style

Stops and reports at the first failing pass.

---

## Editor-setup

Write LSP config files for detected editors.

```
command -> editor-setup
```

Supported: Vim · Neovim · Helix · Emacs. For VS Code: install the extension via the marketplace.

---

## Other commands

| Command | Action |
|---|---|
| `help` | List all commands |
| `update` | Reinstall from latest main |
| `exit` | Quit |

---

# Part 4 — Quick Reference

## Function syntax

```
let name(a: Type, b: Type) -> result: Type {
    pipe;
}

let name(a: Type) {
    pipe;
}
```

## Bullang pipe syntax

```
(a, b)   : a + b              -> {result};
(s)      : builtin::to_upper  -> {upper};
(a, b)   : add                -> {sum};
("hello"): shout              -> {loud};
(x)      : some_fn            -> {};
```

## BullScript pipe syntax

Always typed, no `let`, no functions — a `.busc` file or a prompt line is
just pipes:

```
(a: i64, b: i64) : builtin::add       -> {result: i64};
(s: String)      : builtin::to_upper  -> {upper: String};
(a: i64, b: i64) : bag::double        -> {sum: i64};
(x: i64)         : x * 2              -> {};
```

## All builtins at a glance

Bullang's stdlib (Part 1):
```
Math      abs  pow  powf  sqrt  log  exp  min  max  clamp
Cond      tern
String    to_upper  to_lower  trim  len  starts_with  ends_with
          replace_str  to_string  parse_i64
Algo      swap  insertion_sort  quick_sort  merge_sort  radix_sort
I/O       in  out  open  close  time
System    args  exit  env  sleep
```

BullScript's own fixed `builtin::*` table (Part 2) — separate, not reused
from Bullang's:
```
add  to_upper  to_lower  trim  out  run  capture
```

## Bullscript directives

```
help                     Print the in-prompt help
bag::add <path> <name>   Register a .busc file in the bag
bag::remove <name>       Remove a single bag entry
bag::list                List your bag entries
record::start            Start capturing pipes typed at the prompt
record::end              Stop, preview, optionally save as a bag entry
exit                     Quit (Ctrl+D also works)
```

## Bullarchy commands

```
init <name> [opts]         Scaffold a project
convert <folder|file> [opts]  Transpile
fmt [folder] [--dry-run]   Format .bu files
check                      Validate and type-check
editor-setup               Write LSP config
update                     Reinstall from latest
help / exit
```

## inventory.bu example

```
#rank: tactic;
#lang: rs;

struct Point {
    x : i32,
    y : i32,
}

enum Color {
    Red,
    Green,
    Blue,
}

math   : add, subtract, multiply;
shapes : area, perimeter;
```
