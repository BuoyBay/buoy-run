# Run .bui scripts

`buoy-run` compiles a script into a private temporary directory, runs it, forwards
its exit status and removes the temporary build. It preserves the caller's
working directory, environment and standard input/output.

## Build and run

Install the Buoy compiler, then run from this checkout:

```sh
mkdir -p dist/runner
buoy src/main.bui -o dist/runner/buoy-run
./dist/runner/buoy-run examples/ahoy_hello.bui 'an argument with spaces'
./dist/runner/buoy-run repl
```

Keep compiler-deployed libraries beside the runner. Add its absolute directory to
your `PATH` to invoke it as `buoy-run`.

## Executable scripts

A script can start with `#!/usr/bin/env buoy-run` and be marked executable. The
runner handles that first line before invoking the compiler; direct `buoy --check`
expects a normal Buoy source file without the shebang.

Arguments after the script path are forwarded literally. Identifier imports find
sibling modules, and the nearest ancestor `packages/` directory supplies package
archives. Each invocation recompiles; desktop bundles and custom compiler flags
are outside the runner's scope. See the [usage guide](src/README.md) for imports,
I/O, temporary-resource lifetime and environment overrides.

This implementation uses POSIX utilities/libc and supports macOS and Linux.
`BUOY_RUN_BUOY` selects the compiler executable; `BUOY_RUN_SCRIPT` identifies the
original script path. The former `AHOY_*` variables remain compatibility aliases.
[Ahoy](https://github.com/BuoyBay/ahoy) is the developer toolkit.

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
