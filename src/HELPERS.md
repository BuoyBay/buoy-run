# Workspace helpers through buoy-run

Build the runner once, then use the `.bui` entrypoints from the repository root:

```sh
buoy src/main.bui -o dist/buoy-run
export PATH="$PWD/dist:$PATH"
buoy-run tools/build_calls.bui
```

The Python helpers remain available for compatibility. Their Buoy counterparts
perform build and test orchestration in Buoy. They use the installed compiler,
POSIX utilities, and CouchDB where the original workflow requires it. These
entrypoints are tested on macOS and share buoy-run's POSIX scope; they are not Windows
launchers. Live tests also use `curl` for readiness checks and `openssl` to create
temporary credentials. The 50 HTTP fixture cases live in
`tests/fixtures/couchdb_steps.json`, including binary payloads encoded as hex.

| Existing helper | buoy-run entrypoint | Purpose |
| --- | --- | --- |
| `tools/build_couchdb.py` | `tools/build_couchdb.bui` | Seal and copy the CouchDB package |
| `tools/build_scraping.py` | `tools/build_scraping.bui` | Seal and copy the scraping package |
| `tools/build_calls.py` | `tools/build_calls.bui` | Build packages and the native Calls app |
| `tools/build_docs.py` | `tools/build_docs.bui` | Embed the offline snapshot and build Buoy Docs |
| `tools/port_workspace.py` | `tools/port_workspace.bui` | Create an isolated Calls compile workspace |
| `tests/run_couchdb_tests.py` | `tests/run_couchdb_tests.bui` | Compiler, package, and mock HTTP client checks |
| `tests/run_couchdb_live_tests.py` | `tests/run_couchdb_live_tests.bui` | Isolated live CouchDB checks |
| `tests/run_calls_live_tests.py` | `tests/run_calls_live_tests.bui` | Live application repository and image redirect checks |
| `tests/run_port_tests.py` | `tests/run_port_tests.bui` | Build and run the complete port checks |
| `tests/run_ahoy_tests.py` | `tests/run_ahoy_tests.bui` | Compiler and runner integration checks |
| `tools/index_buoy_docs.cjs` | `tools/index_buoy_docs.bui` | Run the existing local documentation indexer |
| `tools/sync_buoy_docs.cjs` | `tools/sync_buoy_docs.bui` | Run the existing online documentation refresher |
| `tests/docs_viewer_test.cjs` | `tests/docs_viewer_test.bui` | Run the actual viewer JavaScript and DOM tests |
| `tests/scraping_extractor_test.cjs` | `tests/scraping_extractor_test.bui` | Run the actual extractor JavaScript and DOM tests |

## JavaScript engines

The last four entrypoints launch the existing JavaScript helpers with Node. This
preserves the DOM, Markdown rendering, URL handling, and application JavaScript
behavior covered by those helpers. These are Buoy command-line entrypoints, not
native rewrites of the JavaScript engines. Install their dependencies once:

```sh
npm install --ignore-scripts
```

Their standard streams and exit status pass through, and arguments are forwarded
literally. The local indexer accepts an optional documentation directory, resolved
relative to the caller's current directory:

```sh
buoy-run tools/index_buoy_docs.bui /Applications/Buoy/docs
```

Both documentation operations replace `apps/BuoyDocs/snapshot/docs.json`.
`sync_buoy_docs.bui` fetches the website explicitly; building or running the apps
does not trigger that refresh. All four JavaScript-backed entrypoints accept
`--help` without running the underlying engine.

The interactive scratchpad is available with `buoy-run repl`. Its native integration
checks run with `buoy-run tests/run_repl_tests.bui`; see the
[REPL instructions](README.md#interactive-scratchpad).

Test suites and fixtures are maintainer-local and are not part of this public repository.
