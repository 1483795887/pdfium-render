# Examples

Simple examples demonstrating how to use `pdfium-render` on both native and WASM compilation targets.

* `export.rs`: exports the individual pages in `test/export-test.pdf` to JPGs in the working directory. Run this example via `cargo run --example export`. The example will attempt to bind to a copy of Pdfium in the working directory, falling back to the system-bundled library if local loading fails. See comments in the source file for more details.
* `form.rs`: exports the individual pages in `test/form-test.pdf` to JPGs in the working directory. The sample PDF includes pre-filled form fields, the values of which should also be rendered. Run this example via `cargo run --example form`. See comments in the source file for more details.
* `text.rs`: extracts and outputs the text on each page in `test/text-test.pdf` to the console. Run this example via `cargo run --example text`.
* `objects.rs`: outputs information about each page object on each page in `test/export-test.pdf` to the console. Run this example via `cargo run --example objects`.
* `wasm.rs`: demonstrates `pdfium-render` running in a browser. This requires some manual bundling of the correct resources; read on.

## Bundling for WASM

Since `pdfium-render` does not include Pdfium itself, an external pre-packaged WASM build of `pdfium` is required. Suitable builds are available from https://github.com/paulo-coutinho/pdfium-lib/releases.

* Build the WASM module for the sample: `cargo install wasm-pack && wasm-pack build examples/ --target no-modules`. This creates a WASM module and supporting Javascript files in `examples/pkg`.
* Copy the `pdfium_render_wasm_example.js` and `pdfium_render_wasm_example_bg.wasm` files from `examples/pkg/` into a release folder.
* Download a pre-packaged WASM build from https://github.com/paulo-coutinho/pdfium-lib/releases and extract the `release/node/pdfium.js` and `release/node/pdfium.wasm` files into your release folder.
* Copy the `index.html` file from `examples` into your release folder.
* Optionally copy the `serve.sh` file from `examples` into your release folder; this is a tiny script that will spin up a development webserver for you using the `basic-http-server` crate. You can ignore this if you have another way of serving the files.
* Serve the content from your release folder using a webserver or by running `serve.sh`. If you use `serve.sh`, then the content will be available at http://localhost:4000.

You should see the sizes of each individual page in `test/form-test.pdf` logged to the Javascript console, and the first page in the file will be rendered into an HTML canvas element.

Comments in the `index.html` file explain how to instantiate both the compiled Pdfium and the example
WASM modules and bind them together dynamically at run time. The basic recipe is simple:

* Load and instantiate the Pdfium WASM module first.
* Once Pdfium is instantiated, load and instantiate the WASM module for your compiled Rust application.
* Once your WASM module is instantiated, call `pdfium-render`'s exported `initialize_pdfium_render()` function, passing it the Pdfium WASM module.
* You can now call any Pdfium-related functions exported by your compiled Rust application.