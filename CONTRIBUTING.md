# Contributing to buoy-run

This repository owns **buoy-run**, the Buoy script runner and interactive
scratchpad. Source remains under `src/`; the public **Ahoy** name belongs
to the separate developer toolkit. Preserve argument forwarding, working-directory
behavior, cleanup and the documented legacy environment-variable aliases.

## Development conventions

Use the installed Buoy language reference and verify changed `.bui` files with
the compiler. Prefer native Buoy/BuoyScript for new automation. Preserve public
command names, environment-variable contracts and explicit platform limitations.

## Build context

Read the [runner guide](src/README.md). On a supported POSIX host,
with Buoy installed, the runner can be built from this checkout:

```sh
mkdir -p dist/buoy-run
buoy src/main.bui --check --diagnostic-format json
buoy src/main.bui -o dist/buoy-run/buoy-run
./dist/buoy-run/buoy-run examples/ahoy_hello.bui 'argument with spaces'
```

Keep compiler-deployed runtime files with the executable. Check actual subprocess
exit status and script I/O for behavior changes; Windows support must not be
inferred from POSIX checks.

## Verification and pull requests

For each edited Buoy entrypoint, in its documented staged dependency context:

```sh
buoy path/to/entrypoint.bui --check --diagnostic-format json
```

Require exit status zero and an empty `errors` array. Read warnings. Check each
affected conditional platform branch with `--target`; compiler checks do not prove
native runtime behavior on another OS. Build/link and exercise the changed API or
workflow on its supported host as well.

Include your compiler/tool versions, host/target, exact build and
reproduction commands, expected behavior and observed output in the pull request.
Do not commit generated builds, caches, credentials or machine
configuration. Keep the patch focused and note any unverified platforms or runtime
prerequisites. Inventory checks alone are not build or test acceptance.

## License of contributions

Original contributions are accepted under the existing license for the files you
change: `MIT OR Apache-2.0` for first-party code and `CC-BY-4.0` for documentation
prose. Code samples follow their code's license. No CLA or DCO signing step is
required. Keep your own authorship and copyright attribution.

Read [LICENSE](LICENSE) and [LICENSING.md](LICENSING.md). Preserve upstream license,
copyright, attribution and provenance files for third-party material, and identify
its source in the pull request. Submit only material you have the right to license.
Reusable packages and tools keep their permissive licensing boundary; changes to
Studio's AGPL application code belong in its separate repository.
