# buoy-run

A small Buoy script runner written in Buoy. It compiles a script into a private
temporary directory, runs it, returns its exit code, and removes the temporary
build. Requires the Buoy compiler on PATH. Built and tested on macOS and Linux; uses POSIX
shell utilities and libc APIs, so this version does not support Windows.

From the repository root:

```sh
buoy src/main.bui --check --diagnostic-format json
buoy src/main.bui -o dist/buoy-run
./dist/buoy-run examples/ahoy_hello.bui 'an argument with spaces'
```

To use `buoy-run` by name in the current terminal:

```sh
export PATH="$PWD/dist:$PATH"
buoy-run examples/ahoy_hello.bui
```

For permanent use, put the absolute path to this repository's `dist` directory in
your shell's PATH configuration. The compiler also needs to stay installed.
`BUOY_RUN_BUOY` can select a different compiler executable (a path, without flags).

## Standard input and output

The script inherits stdin, stdout, stderr, environment, and working directory.
Use `Print`, `StdOut.Write`, `StdOut.WriteLine`, `StdErr.WriteLine`, and
`StdIn.ReadLine` normally. Flush stdout after a prompt without a newline;
`StdIn.ReadLine()` returns an Optional, with no value at EOF.

```sh
printf 'Sid\n' | buoy-run examples/ahoy_hello.bui
buoy-run examples/ahoy_hello.bui < names.txt > greetings.txt 2> errors.txt
```

Program arguments after the script path are forwarded literally, including empty
arguments, quotes, spaces, and shell metacharacters. As with other compiled Buoy
programs, `args(0)` is the temporary executable's path; user arguments start at 1.
The caller's working directory is preserved, so relative data paths use that
directory. `BUOY_RUN_SCRIPT` (also exported as `AHOY_SCRIPT` for compatibility) contains the original source file's absolute path, so
scripts can find sibling data without relying on the temporary executable path. There is no run timeout or output capture.

## Executable scripts

Add `#!/usr/bin/env buoy-run` as the very first line of a Buoy program, then mark the
file executable with `chmod +x script.bui`. You can then run `./script.bui`.
For example, prepend that line to a copy of `examples/ahoy_hello.bui`.

The runner comments out the shebang in a temporary copy, keeping line numbers
unchanged. The original file stays untouched. Identifier imports such as
`Import helper` are searched in the original source directory, and the nearest
ancestor's `packages` directory is made available to the compiler. Relative
quoted-path imports resolve from the temporary source directory; use an
identifier import for a sibling module or an absolute quoted path instead.
Diagnostics for shebang scripts name the
temporary source. Buoy itself still rejects the shebang, so use `buoy-run` to run
these scripts and omit the directive when using `buoy --check` directly.

Each invocation recompiles; there is no executable cache. Desktop app bundles and
project-specific build flags/manifests are outside this small console runner's
scope. Temporary executable resources disappear when the invocation finishes.

## Project helpers

Use the runner for your own Buoy automation entrypoints. Keep helper source,
configuration and assets in the project and pass their paths explicitly.
[Helper usage](HELPERS.md) describes arguments, imports and external engines.

Scripts importing `helper_runtime` use an isolated source tree for shared helper
modules. Data/output paths still resolve against the original workspace through
`BUOY_RUN_SCRIPT` (with `AHOY_SCRIPT` retained as an alias).

## Interactive scratchpad

Start with `buoy-run repl` (or `./dist/buoy-run repl` from the repository root).
Enter a statement such as `Print "hello"`, or use `? 6 * 7` to print an expression.
Each submission is compiled and runs once in a fresh process. Variables and
objects belong to that submission. Compilation takes a moment; this mode uses the
installed Buoy compiler rather than an interpreter or JIT.

Multiline input uses Buoy's keywords automatically. Start `If … Then`, `For`,
`While`, `Do`, `Select Case`, or `Try`, and the prompt changes to `...` until the
matching `End If`, `Next`, `End While`/`Wend`, `Loop`, `End Select`, or `End Try`.
Nested blocks, comments, quoted strings, single-line `If`, and `_` continuations
are handled. Blank lines inside a block do not execute it.

Function, Sub, Class, Interface, Structure, Enum, and Namespace declarations also
collect through their normal closing keywords. Completed declarations, imports,
and constants are compiler-checked and remembered as source for later submissions.
Use `:defs` to inspect them and `:reset` to clear them. Invalid declarations are
rejected without changing the accepted definitions. A declaration named `Main`
runs as a complete program instead of being remembered.

Each executable submission still runs in a fresh process. Remembered definitions
are prepended to the new source; previous executable submissions are never
replayed automatically. Runtime objects and local variables do not carry over.
Shared/property initializers in remembered declarations can run in each new
process according to normal Buoy semantics.

Use `:begin` / `:end` when several independent statements should run together—for
example, a `Dim` followed by statements that use that variable. Use `:program` /
`:end` to enter a complete program manually or as a fallback for advanced syntax
that the lexical boundary tracker does not recognize. In explicit mode, closing
Buoy keywords do not trigger execution before `:end`. The real compiler validates
every submission; boundary tracking does not replace parsing or type checking.
Identifier imports search the working directory, and the nearest ancestor's
`packages/` is available. Relative quoted-path imports resolve from the temporary
submission file, so use an absolute quoted path when needed.

Commands:

- `:help` shows commands; `:quit` or Ctrl-D exits.
- `:cancel` discards a block being entered.
- `:show` prints the current block or the last successful executable submission.
- `:defs` prints remembered declarations.
- `:run` explicitly reruns the last successful executable submission, including
  its effects, using the current definitions.
- `:reset` clears definitions, the remembered submission, and any pending block.

Compiler errors leave the session open. A program's nonzero exit status is
reported on stderr. An unfinished block is discarded at EOF. Prompts are on stderr
and appear only when stdin is a terminal; piped sessions produce only program
output on stdout. Interactive submissions can read from the terminal; avoid
mixing a piped command session with program stdin reads, since both consume the
same input stream. Ctrl-C interrupts a running program; use `:quit` to finish the
session normally.

The former Ahoy script runner is now `buoy-run`; Ahoy names the developer
toolkit. The source layout and the `AHOY_BUOY`, `AHOY_PACKAGES`,
and `AHOY_SCRIPT` environment variables remain supported for compatibility.
