# Run project automation with buoy-run

`buoy-run` compiles and executes a Buoy source file while preserving the caller's
working directory, environment, arguments and standard streams. Use it for
project automation written in Buoy.

Build the runner from this checkout:

```sh
mkdir -p dist/runner
buoy src/main.bui -o dist/runner/buoy-run
export PATH="$PWD/dist/runner:$PATH"
buoy-run examples/ahoy_hello.bui 'an argument with spaces'
```

The example prompts for a name on standard input. Keep the compiler-deployed
runtime libraries beside the runner.

## Running your scripts

Provide a `.bui` entrypoint and pass its path followed by its arguments:

```sh
buoy-run /absolute/path/to/project-script.bui --help
```

The script receives the arguments after its filename. Imports can use sibling
modules and the nearest ancestor `packages/` directory. Choose an installed
compiler with `BUOY_RUN_BUOY`; the former `AHOY_BUOY` variable remains an alias.
`BUOY_RUN_SCRIPT` identifies the original script path for helpers that need it.

Scripts that start other programs retain those programs' dependencies. For
example, a script that invokes Node needs Node and its own JavaScript dependency
installation. The runner does not convert external engines into native code.

Use `buoy-run repl` for the interactive scratchpad. See the
[runner guide](README.md#interactive-scratchpad) for input and reset commands.
