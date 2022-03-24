<!-- Generated with Stardoc: http://skydoc.bazel.build -->

# Crate Universe

Crate Universe is a set of Bazel rule for generating Rust targets using Cargo.

## Experimental

`crate_universe` is experimental, and may have breaking API changes at any time. These instructions may also change without notice.

## Setup

After loading `rules_rust` in your workspace, set the following to begin using `crate_universe`:

```python
load("@rules_rust//crate_universe:repositories.bzl", "crate_universe_dependencies")

crate_universe_dependencies()
```

Note that if the current version of `rules_rust` is not a release artifact, you may need to set additional
flags such as [`bootstrap = True`](#crate_universe_dependencies-bootstrap) on the `crate_universe_dependencies`
call above or [crates_repository::generator_urls](#crates_repository-generator_urls) in uses of `crates_repository`.

## Rules

- [crate_universe_dependencies](#crate_universe_dependencies)
- [crates_repository](#crates_repository)
- [crates_vendor](#crates_vendor)
- [crate.annotation](#crateannotation)
- [crate.spec](#cratespec)
- [crate.workspace_member](#crateworkspace_member)
- [render_config](#render_config)
- [splicing_config](#splicing_config)

## `crates_repository` Workflows

The [`crates_repository`](#crates_repository) rule (the primary repository rule of `cargo-bazel`) supports a number of different ways users
can express and organize their dependencies. The most common are listed below though there are more to be found in
the [./examples/crate_universe](https://github.com/bazelbuild/rules_rust/tree/main/examples/crate_universe) directory.

### Cargo Workspaces

One of the simpler ways to wire up dependencies would be to first structure your project into a [Cargo workspace][cw].
The `crates_repository` rule can ingest a root `Cargo.toml` file and generate dependencies from there.

```python
load("@rules_rust//crate_universe:defs.bzl", "crates_repository")

crates_repository(
    name = "crate_index",
    lockfile = "//:Cargo.Bazel.lock",
    manifests = ["//:Cargo.toml"],
)

load("@crate_index//:defs.bzl", "crate_repositories")

crate_repositories()
```

The generated `crates_repository` contains helper macros which make collecting dependencies for Bazel targets simpler.
Notably, the `all_crate_deps` and `aliases` macros commonly allow the `Cargo.toml` files to be the single source of
truth for dependencies. Since these macros come from the generated repository, the dependencies and alias definitions
they return will automatically update BUILD targets.

```python
load("@crate_index//:defs.bzl", "aliases", "all_crate_deps")
load("@rules_rust//rust:defs.bzl", "rust_library", "rust_test")

rust_library(
    name = "lib",
    aliases = aliases(),
    deps = all_crate_deps(
        normal = True,
    ),
    proc_macro_deps = all_crate_deps(
        proc_macro = True,
    ),
)

rust_test(
    name = "unit_test",
    crate = ":lib",
    aliases = aliases(
        normal_dev = True,
        proc_macro_dev = True,
    ),
    deps = all_crate_deps(
        normal_dev = True,
    ),
    proc_macro_deps = all_crate_deps(
        proc_macro_dev = True,
    ),
)
```

### Direct Packages

In cases where Rust targets have heavy interractions with other Bazel targests ([Cc][cc], [Proto][proto], etc.),
maintaining `Cargo.toml` files may have deminishing returns as things like [rust-analyzer][ra] begin to be confused
about missing targets or environment variables defined only in Bazel. In workspaces like this, it may be desirable
to have a "Cargo free" setup. `crates_repository` supports this through the `packages` attribute.

```python
load("@cargo_bazel//:defs.bzl", "crate", "crates_repository", "render_config")

crates_repository(
    name = "crate_index",
    lockfile = "//:Cargo.Bazel.lock",
    packages = {
        "async-trait": crate.spec(
            version = "0.1.51",
        ),
        "mockall": crate.spec(
            version = "0.10.2",
        ),
        "tokio": crate.spec(
            version = "1.12.0",
        ),
    },
    # Setting the default package name to `""` forces the use of the macros defined in this repository
    # to always use the root package when looking for dependencies or aliases. This should be considered
    # optional as the repository also exposes alises for easy access to all dependencies.
    render_config = render_config(
        default_package_name = ""
    ),
)

load("@crate_index//:defs.bzl", "crate_repositories")

crate_repositories()
```

Consuming dependencies may be more ergonomic in this case through the aliases defined in the new repository.

```python
load("@rules_rust//rust:defs.bzl", "rust_library", "rust_test")

rust_library(
    name = "lib",
    deps = [
        "@crate_index//:tokio",
    ],
    proc_macro_deps = [
        "@crate_index//:async-trait",
    ],
)

rust_test(
    name = "unit_test",
    crate = ":lib",
    deps = [
        "@crate_index//:mockall",
    ],
)
```

[cw]: https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html
[cc]: https://docs.bazel.build/versions/main/be/c-cpp.html
[proto]: https://rules-proto-grpc.com/en/latest/lang/rust.html
[ra]: https://rust-analyzer.github.io/



<a id="#crates_repository"></a>

## crates_repository

<pre>
crates_repository(<a href="#crates_repository-name">name</a>, <a href="#crates_repository-annotations">annotations</a>, <a href="#crates_repository-cargo_config">cargo_config</a>, <a href="#crates_repository-extra_workspace_member_url_template">extra_workspace_member_url_template</a>,
                  <a href="#crates_repository-extra_workspace_members">extra_workspace_members</a>, <a href="#crates_repository-generate_build_scripts">generate_build_scripts</a>, <a href="#crates_repository-generator">generator</a>, <a href="#crates_repository-generator_sha256s">generator_sha256s</a>,
                  <a href="#crates_repository-generator_urls">generator_urls</a>, <a href="#crates_repository-isolated">isolated</a>, <a href="#crates_repository-lockfile">lockfile</a>, <a href="#crates_repository-lockfile_kind">lockfile_kind</a>, <a href="#crates_repository-manifests">manifests</a>, <a href="#crates_repository-packages">packages</a>, <a href="#crates_repository-quiet">quiet</a>,
                  <a href="#crates_repository-render_config">render_config</a>, <a href="#crates_repository-repo_mapping">repo_mapping</a>, <a href="#crates_repository-rust_toolchain_cargo_template">rust_toolchain_cargo_template</a>,
                  <a href="#crates_repository-rust_toolchain_rustc_template">rust_toolchain_rustc_template</a>, <a href="#crates_repository-rust_version">rust_version</a>, <a href="#crates_repository-splicing_config">splicing_config</a>,
                  <a href="#crates_repository-supported_platform_triples">supported_platform_triples</a>)
</pre>

A rule for defining and downloading Rust dependencies (crates).

Environment Variables:

| variable | usage |
| --- | --- |
| `CARGO_BAZEL_GENERATOR_SHA256` | The sha256 checksum of the file located at `CARGO_BAZEL_GENERATOR_URL` |
| `CARGO_BAZEL_GENERATOR_URL` | The URL of a cargo-bazel binary. This variable takes precedence over attributes and can use `file://` for local paths |
| `CARGO_BAZEL_ISOLATED` | An authorative flag as to whether or not the `CARGO_HOME` environment variable should be isolated from the host configuration |
| `CARGO_BAZEL_REPIN` | An indicator that the dependencies represented by the rule should be regenerated. `REPIN` may also be used. |



**ATTRIBUTES**


| Name  | Description | Type | Mandatory | Default |
| :------------- | :------------- | :------------- | :------------- | :------------- |
| <a id="crates_repository-name"></a>name |  A unique name for this repository.   | <a href="https://bazel.build/docs/build-ref.html#name">Name</a> | required |  |
| <a id="crates_repository-annotations"></a>annotations |  Extra settings to apply to crates. See [crate.annotations](#crateannotations).   | <a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> List of strings</a> | optional | {} |
| <a id="crates_repository-cargo_config"></a>cargo_config |  A [Cargo configuration](https://doc.rust-lang.org/cargo/reference/config.html) file   | <a href="https://bazel.build/docs/build-ref.html#labels">Label</a> | optional | None |
| <a id="crates_repository-extra_workspace_member_url_template"></a>extra_workspace_member_url_template |  The registry url to use when fetching extra workspace members   | String | optional | "https://crates.io/api/v1/crates/{name}/{version}/download" |
| <a id="crates_repository-extra_workspace_members"></a>extra_workspace_members |  Additional crates to download and include as a workspace member. This is unfortunately required in order to add information about "binary-only" crates so that a <code>rust_binary</code> may be generated for it. [rust-lang/cargo#9096](https://github.com/rust-lang/cargo/issues/9096) tracks an RFC which may solve for this.   | <a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a> | optional | {} |
| <a id="crates_repository-generate_build_scripts"></a>generate_build_scripts |  Whether or not to generate [cargo build scripts](https://doc.rust-lang.org/cargo/reference/build-scripts.html) by default.   | Boolean | optional | True |
| <a id="crates_repository-generator"></a>generator |  The absolute label of a generator. Eg. <code>@cargo_bazel_bootstrap//:cargo-bazel</code>. This is typically used when bootstrapping   | String | optional | "" |
| <a id="crates_repository-generator_sha256s"></a>generator_sha256s |  Dictionary of <code>host_triple</code> -&gt; <code>sha256</code> for a <code>cargo-bazel</code> binary.   | <a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a> | optional | {} |
| <a id="crates_repository-generator_urls"></a>generator_urls |  URL template from which to download the <code>cargo-bazel</code> binary. <code>{host_triple}</code> and will be filled in according to the host platform.   | <a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a> | optional | {} |
| <a id="crates_repository-isolated"></a>isolated |  If true, <code>CARGO_HOME</code> will be overwritten to a directory within the generated repository in order to prevent other uses of Cargo from impacting having any effect on the generated targets produced by this rule. For users who either have multiple <code>crate_repository</code> definitions in a WORKSPACE or rapidly re-pin dependencies, setting this to false may improve build times. This variable is also controled by <code>CARGO_BAZEL_ISOLATED</code> environment variable.   | Boolean | optional | True |
| <a id="crates_repository-lockfile"></a>lockfile |  The path to a file to use for reproducible renderings. Two kinds of lock files are supported, Cargo (<code>Cargo.lock</code> files) and Bazel (custom files generated by this rule, naming is irrelevant). Bazel lockfiles should be the prefered kind as they're desigend with Bazel's notions of reporducibility in mind. Cargo lockfiles can be used in cases where it's intended to be the source of truth, but more work will need to be done to generate BUILD files which are not guaranteed to be determinsitic.   | <a href="https://bazel.build/docs/build-ref.html#labels">Label</a> | required |  |
| <a id="crates_repository-lockfile_kind"></a>lockfile_kind |  Two different kinds of lockfiles are supported, the custom "Bazel" lockfile, which is generated by this rule, and Cargo lockfiles (<code>Cargo.lock</code>). This attribute allows for explicitly defining the type in cases where it may not be auto-detectable.   | String | optional | "auto" |
| <a id="crates_repository-manifests"></a>manifests |  A list of Cargo manifests (<code>Cargo.toml</code> files).   | <a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a> | optional | [] |
| <a id="crates_repository-packages"></a>packages |  A set of crates (packages) specifications to depend on. See [crate.spec](#crate.spec).   | <a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a> | optional | {} |
| <a id="crates_repository-quiet"></a>quiet |  If stdout and stderr should not be printed to the terminal.   | Boolean | optional | True |
| <a id="crates_repository-render_config"></a>render_config |  The configuration flags to use for rendering. Use <code>//crate_universe:defs.bzl\%render_config</code> to generate the value for this field. If unset, the defaults defined there will be used.   | String | optional | "" |
| <a id="crates_repository-repo_mapping"></a>repo_mapping |  A dictionary from local repository name to global repository name. This allows controls over workspace dependency resolution for dependencies of this repository.&lt;p&gt;For example, an entry <code>"@foo": "@bar"</code> declares that, for any time this repository depends on <code>@foo</code> (such as a dependency on <code>@foo//some:target</code>, it should actually resolve that dependency within globally-declared <code>@bar</code> (<code>@bar//some:target</code>).   | <a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a> | required |  |
| <a id="crates_repository-rust_toolchain_cargo_template"></a>rust_toolchain_cargo_template |  The template to use for finding the host <code>cargo</code> binary. <code>{version}</code> (eg. '1.53.0'), <code>{triple}</code> (eg. 'x86_64-unknown-linux-gnu'), <code>{arch}</code> (eg. 'aarch64'), <code>{vendor}</code> (eg. 'unknown'), <code>{system}</code> (eg. 'darwin'), <code>{cfg}</code> (eg. 'exec'), and <code>{tool}</code> (eg. 'rustc.exe') will be replaced in the string if present.   | String | optional | "@rust_{system}_{arch}//:bin/{tool}" |
| <a id="crates_repository-rust_toolchain_rustc_template"></a>rust_toolchain_rustc_template |  The template to use for finding the host <code>rustc</code> binary. <code>{version}</code> (eg. '1.53.0'), <code>{triple}</code> (eg. 'x86_64-unknown-linux-gnu'), <code>{arch}</code> (eg. 'aarch64'), <code>{vendor}</code> (eg. 'unknown'), <code>{system}</code> (eg. 'darwin'), <code>{cfg}</code> (eg. 'exec'), and <code>{tool}</code> (eg. 'cargo.exe') will be replaced in the string if present.   | String | optional | "@rust_{system}_{arch}//:bin/{tool}" |
| <a id="crates_repository-rust_version"></a>rust_version |  The version of Rust the currently registered toolchain is using. Eg. <code>1.56.0</code>, or <code>nightly-2021-09-08</code>   | String | optional | "1.59.0" |
| <a id="crates_repository-splicing_config"></a>splicing_config |  The configuration flags to use for splicing Cargo maniests. Use <code>//crate_universe:defs.bzl\%rsplicing_config</code> to generate the value for this field. If unset, the defaults defined there will be used.   | String | optional | "" |
| <a id="crates_repository-supported_platform_triples"></a>supported_platform_triples |  A set of all platform triples to consider when generating dependencies.   | List of strings | optional | ["i686-apple-darwin", "i686-pc-windows-msvc", "i686-unknown-linux-gnu", "x86_64-apple-darwin", "x86_64-pc-windows-msvc", "x86_64-unknown-linux-gnu", "aarch64-apple-darwin", "aarch64-apple-ios", "aarch64-linux-android", "aarch64-unknown-linux-gnu", "arm-unknown-linux-gnueabi", "armv7-unknown-linux-gnueabi", "i686-linux-android", "i686-unknown-freebsd", "powerpc-unknown-linux-gnu", "s390x-unknown-linux-gnu", "wasm32-unknown-unknown", "wasm32-wasi", "x86_64-apple-ios", "x86_64-linux-android", "x86_64-unknown-freebsd"] |


<a id="#crates_vendor"></a>

## crates_vendor

<pre>
crates_vendor(<a href="#crates_vendor-name">name</a>, <a href="#crates_vendor-annotations">annotations</a>, <a href="#crates_vendor-buildifier">buildifier</a>, <a href="#crates_vendor-cargo_bazel">cargo_bazel</a>, <a href="#crates_vendor-generate_build_scripts">generate_build_scripts</a>, <a href="#crates_vendor-manifests">manifests</a>, <a href="#crates_vendor-mode">mode</a>,
              <a href="#crates_vendor-packages">packages</a>, <a href="#crates_vendor-repository_name">repository_name</a>, <a href="#crates_vendor-splicing_config">splicing_config</a>, <a href="#crates_vendor-supported_platform_triples">supported_platform_triples</a>, <a href="#crates_vendor-vendor_path">vendor_path</a>)
</pre>

A rule for defining Rust dependencies (crates) and writing targets for them to the current workspace

**ATTRIBUTES**


| Name  | Description | Type | Mandatory | Default |
| :------------- | :------------- | :------------- | :------------- | :------------- |
| <a id="crates_vendor-name"></a>name |  A unique name for this target.   | <a href="https://bazel.build/docs/build-ref.html#name">Name</a> | required |  |
| <a id="crates_vendor-annotations"></a>annotations |  Extra settings to apply to crates. See [crate.annotations](#crateannotations).   | <a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> List of strings</a> | optional | {} |
| <a id="crates_vendor-buildifier"></a>buildifier |  The path to a [buildifier](https://github.com/bazelbuild/buildtools/blob/5.0.1/buildifier/README.md) binary used to format generated BUILD files.   | <a href="https://bazel.build/docs/build-ref.html#labels">Label</a> | optional | //crate_universe/private/vendor:buildifier |
| <a id="crates_vendor-cargo_bazel"></a>cargo_bazel |  The cargo-bazel binary to use for vendoring. If this attribute is not set, then a <code>CARGO_BAZEL_GENERATOR_PATH</code> action env will be used.   | <a href="https://bazel.build/docs/build-ref.html#labels">Label</a> | optional | @cargo_bazel_bootstrap//:binary |
| <a id="crates_vendor-generate_build_scripts"></a>generate_build_scripts |  Whether or not to generate [cargo build scripts](https://doc.rust-lang.org/cargo/reference/build-scripts.html) by default.   | Boolean | optional | True |
| <a id="crates_vendor-manifests"></a>manifests |  A list of Cargo manifests (<code>Cargo.toml</code> files).   | <a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a> | optional | [] |
| <a id="crates_vendor-mode"></a>mode |  Flags determining how crates should be vendored. <code>local</code> is where crate source and BUILD files are written to the repository. <code>remote</code> is where only BUILD files are written and repository rules used to fetch source code.   | String | optional | "remote" |
| <a id="crates_vendor-packages"></a>packages |  A set of crates (packages) specifications to depend on. See [crate.spec](#crate.spec).   | <a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a> | optional | {} |
| <a id="crates_vendor-repository_name"></a>repository_name |  The name of the repository to generate for <code>remote</code> vendor modes. If unset, the label name will be used   | String | optional | "" |
| <a id="crates_vendor-splicing_config"></a>splicing_config |  The configuration flags to use for splicing Cargo maniests. Use <code>//crate_universe:defs.bzl\%rsplicing_config</code> to generate the value for this field. If unset, the defaults defined there will be used.   | String | optional | "" |
| <a id="crates_vendor-supported_platform_triples"></a>supported_platform_triples |  A set of all platform triples to consider when generating dependencies.   | List of strings | optional | ["i686-apple-darwin", "i686-pc-windows-msvc", "i686-unknown-linux-gnu", "x86_64-apple-darwin", "x86_64-pc-windows-msvc", "x86_64-unknown-linux-gnu", "aarch64-apple-darwin", "aarch64-apple-ios", "aarch64-linux-android", "aarch64-unknown-linux-gnu", "arm-unknown-linux-gnueabi", "armv7-unknown-linux-gnueabi", "i686-linux-android", "i686-unknown-freebsd", "powerpc-unknown-linux-gnu", "s390x-unknown-linux-gnu", "wasm32-unknown-unknown", "wasm32-wasi", "x86_64-apple-ios", "x86_64-linux-android", "x86_64-unknown-freebsd"] |
| <a id="crates_vendor-vendor_path"></a>vendor_path |  The path to a directory to write files into. Absolute paths will be treated as relative to the workspace root   | String | optional | "crates" |


<a id="#crate.spec"></a>

## crate.spec

<pre>
crate.spec(<a href="#crate.spec-package">package</a>, <a href="#crate.spec-version">version</a>, <a href="#crate.spec-default_features">default_features</a>, <a href="#crate.spec-features">features</a>, <a href="#crate.spec-git">git</a>, <a href="#crate.spec-rev">rev</a>)
</pre>

A constructor for a crate dependency.

See [specifying dependencies][sd] in the Cargo book for more details.

[sd]: https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html


**PARAMETERS**


| Name  | Description | Default Value |
| :------------- | :------------- | :------------- |
| <a id="crate.spec-package"></a>package |  The explicit name of the package (used when attempting to alias a crate).   |  <code>None</code> |
| <a id="crate.spec-version"></a>version |  The exact version of the crate. Cannot be used with <code>git</code>.   |  <code>None</code> |
| <a id="crate.spec-default_features"></a>default_features |  Maps to the <code>default-features</code> flag.   |  <code>True</code> |
| <a id="crate.spec-features"></a>features |  A list of features to use for the crate   |  <code>[]</code> |
| <a id="crate.spec-git"></a>git |  The Git url to use for the crate. Cannot be used with <code>version</code>.   |  <code>None</code> |
| <a id="crate.spec-rev"></a>rev |  The git revision of the remote crate. Tied with the <code>git</code> param.   |  <code>None</code> |

**RETURNS**

string: A json encoded string of all inputs


<a id="#crate.annotation"></a>

## crate.annotation

<pre>
crate.annotation(<a href="#crate.annotation-version">version</a>, <a href="#crate.annotation-additive_build_file">additive_build_file</a>, <a href="#crate.annotation-additive_build_file_content">additive_build_file_content</a>, <a href="#crate.annotation-build_script_data">build_script_data</a>,
                 <a href="#crate.annotation-build_script_data_glob">build_script_data_glob</a>, <a href="#crate.annotation-build_script_deps">build_script_deps</a>, <a href="#crate.annotation-build_script_env">build_script_env</a>,
                 <a href="#crate.annotation-build_script_proc_macro_deps">build_script_proc_macro_deps</a>, <a href="#crate.annotation-build_script_rustc_env">build_script_rustc_env</a>, <a href="#crate.annotation-compile_data">compile_data</a>,
                 <a href="#crate.annotation-compile_data_glob">compile_data_glob</a>, <a href="#crate.annotation-crate_features">crate_features</a>, <a href="#crate.annotation-data">data</a>, <a href="#crate.annotation-data_glob">data_glob</a>, <a href="#crate.annotation-deps">deps</a>, <a href="#crate.annotation-gen_build_script">gen_build_script</a>,
                 <a href="#crate.annotation-patch_args">patch_args</a>, <a href="#crate.annotation-patch_tool">patch_tool</a>, <a href="#crate.annotation-patches">patches</a>, <a href="#crate.annotation-proc_macro_deps">proc_macro_deps</a>, <a href="#crate.annotation-rustc_env">rustc_env</a>, <a href="#crate.annotation-rustc_env_files">rustc_env_files</a>,
                 <a href="#crate.annotation-rustc_flags">rustc_flags</a>, <a href="#crate.annotation-shallow_since">shallow_since</a>)
</pre>

A collection of extra attributes and settings for a particular crate. See [crate_repository's annotations arg](#crates_repository-annotations).

**PARAMETERS**


| Name  | Description | Default Value |
| :------------- | :------------- | :------------- |
| <a id="crate.annotation-version"></a>version |  The version or semver-conditions to match with a crate.   |  <code>"*"</code> |
| <a id="crate.annotation-additive_build_file"></a>additive_build_file |  A file containing extra contents to write to the bottom of generated BUILD files.   |  <code>None</code> |
| <a id="crate.annotation-additive_build_file_content"></a>additive_build_file_content |  Extra contents to write to the bottom of generated BUILD files.   |  <code>None</code> |
| <a id="crate.annotation-build_script_data"></a>build_script_data |  A list of labels to add to a crate's <code>cargo_build_script::data</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-build_script_data_glob"></a>build_script_data_glob |  A list of glob patterns to add to a crate's <code>cargo_build_script::data</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-build_script_deps"></a>build_script_deps |  A list of labels to add to a crate's <code>cargo_build_script::deps</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-build_script_env"></a>build_script_env |  Additional environment variables to set on a crate's <code>cargo_build_script::env</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-build_script_proc_macro_deps"></a>build_script_proc_macro_deps |  A list of labels to add to a crate's <code>cargo_build_script::proc_macro_deps</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-build_script_rustc_env"></a>build_script_rustc_env |  Additional environment variables to set on a crate's <code>cargo_build_script::env</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-compile_data"></a>compile_data |  A list of labels to add to a crate's <code>rust_library::compile_data</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-compile_data_glob"></a>compile_data_glob |  A list of glob patterns to add to a crate's <code>rust_library::compile_data</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-crate_features"></a>crate_features |  A list of strings to add to a crate's <code>rust_library::crate_features</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-data"></a>data |  A list of labels to add to a crate's <code>rust_library::data</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-data_glob"></a>data_glob |  A list of glob patterns to add to a crate's <code>rust_library::data</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-deps"></a>deps |  A list of labels to add to a crate's <code>rust_library::deps</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-gen_build_script"></a>gen_build_script |  An authorative flag to determine whether or not to produce <code>cargo_build_script</code> targets for the current crate.   |  <code>None</code> |
| <a id="crate.annotation-patch_args"></a>patch_args |  The <code>patch_args</code> attribute of a Bazel repository rule. See [http_archive.patch_args](https://docs.bazel.build/versions/main/repo/http.html#http_archive-patch_args)   |  <code>None</code> |
| <a id="crate.annotation-patch_tool"></a>patch_tool |  The <code>patch_tool</code> attribute of a Bazel repository rule. See [http_archive.patch_tool](https://docs.bazel.build/versions/main/repo/http.html#http_archive-patch_tool)   |  <code>None</code> |
| <a id="crate.annotation-patches"></a>patches |  The <code>patches</code> attribute of a Bazel repository rule. See [http_archive.patches](https://docs.bazel.build/versions/main/repo/http.html#http_archive-patches)   |  <code>None</code> |
| <a id="crate.annotation-proc_macro_deps"></a>proc_macro_deps |  A list of labels to add to a crate's <code>rust_library::proc_macro_deps</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-rustc_env"></a>rustc_env |  Additional variables to set on a crate's <code>rust_library::rustc_env</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-rustc_env_files"></a>rustc_env_files |  A list of labels to set on a crate's <code>rust_library::rustc_env_files</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-rustc_flags"></a>rustc_flags |  A list of strings to set on a crate's <code>rust_library::rustc_flags</code> attribute.   |  <code>None</code> |
| <a id="crate.annotation-shallow_since"></a>shallow_since |  An optional timestamp used for crates originating from a git repository instead of a crate registry. This flag optimizes fetching the source code.   |  <code>None</code> |

**RETURNS**

string: A json encoded string containing the specified version and separately all other inputs.


<a id="#crate.workspace_member"></a>

## crate.workspace_member

<pre>
crate.workspace_member(<a href="#crate.workspace_member-version">version</a>, <a href="#crate.workspace_member-sha256">sha256</a>)
</pre>

Define information for extra workspace members

**PARAMETERS**


| Name  | Description | Default Value |
| :------------- | :------------- | :------------- |
| <a id="crate.workspace_member-version"></a>version |  The semver of the crate to download. Must be an exact version.   |  none |
| <a id="crate.workspace_member-sha256"></a>sha256 |  The sha256 checksum of the <code>.crate</code> file.   |  <code>None</code> |

**RETURNS**

string: A json encoded string of all inputs


<a id="#crate_universe_dependencies"></a>

## crate_universe_dependencies

<pre>
crate_universe_dependencies(<a href="#crate_universe_dependencies-rust_version">rust_version</a>, <a href="#crate_universe_dependencies-bootstrap">bootstrap</a>)
</pre>

Define dependencies of the `cargo-bazel` Rust target

**PARAMETERS**


| Name  | Description | Default Value |
| :------------- | :------------- | :------------- |
| <a id="crate_universe_dependencies-rust_version"></a>rust_version |  The version of rust to use when generating dependencies.   |  <code>"1.59.0"</code> |
| <a id="crate_universe_dependencies-bootstrap"></a>bootstrap |  If true, a <code>cargo_bootstrap_repository</code> target will be generated.   |  <code>False</code> |


<a id="#render_config"></a>

## render_config

<pre>
render_config(<a href="#render_config-build_file_template">build_file_template</a>, <a href="#render_config-crate_label_template">crate_label_template</a>, <a href="#render_config-crate_repository_template">crate_repository_template</a>,
              <a href="#render_config-crates_module_template">crates_module_template</a>, <a href="#render_config-default_package_name">default_package_name</a>, <a href="#render_config-platforms_template">platforms_template</a>, <a href="#render_config-vendor_mode">vendor_mode</a>)
</pre>

Various settings used to configure rendered outputs

The template parameters each support a select number of format keys. A description of each key
can be found below where the supported keys for each template can be found in the parameter docs

| key | definition |
| --- | --- |
| `name` | The name of the crate. Eg `tokio` |
| `repository` | The rendered repository name for the crate. Directly relates to `crate_repository_template`. |
| `triple` | A platform triple. Eg `x86_64-unknown-linux-gnu` |
| `version` | The crate version. Eg `1.2.3` |
| `target` | The library or binary target of the crate |
| `file` | The basename of a file |


**PARAMETERS**


| Name  | Description | Default Value |
| :------------- | :------------- | :------------- |
| <a id="render_config-build_file_template"></a>build_file_template |  The base template to use for BUILD file names. The available format keys are [<code>{name}</code>, {version}<code>].   |  <code>"//:BUILD.{name}-{version}.bazel"</code> |
| <a id="render_config-crate_label_template"></a>crate_label_template |  The base template to use for crate labels. The available format keys are [<code>{repository}</code>, <code>{name}</code>, <code>{version}</code>, <code>{target}</code>].   |  <code>"@{repository}__{name}-{version}//:{target}"</code> |
| <a id="render_config-crate_repository_template"></a>crate_repository_template |  The base template to use for Crate label repository names. The available format keys are [<code>{repository}</code>, <code>{name}</code>, <code>{version}</code>].   |  <code>"{repository}__{name}-{version}"</code> |
| <a id="render_config-crates_module_template"></a>crates_module_template |  The pattern to use for the <code>defs.bzl</code> and <code>BUILD.bazel</code> file names used for the crates module. The available format keys are [<code>{file}</code>].   |  <code>"//:{file}"</code> |
| <a id="render_config-default_package_name"></a>default_package_name |  The default package name to in the rendered macros. This affects the auto package detection of things like <code>all_crate_deps</code>.   |  <code>None</code> |
| <a id="render_config-platforms_template"></a>platforms_template |  The base template to use for platform names. See [platforms documentation](https://docs.bazel.build/versions/main/platforms.html). The available format keys are [<code>{triple}</code>].   |  <code>"@rules_rust//rust/platform:{triple}"</code> |
| <a id="render_config-vendor_mode"></a>vendor_mode |  An optional configuration for rendirng content to be rendered into repositories.   |  <code>None</code> |

**RETURNS**

string: A json encoded struct to match the Rust `config::RenderConfig` struct


<a id="#splicing_config"></a>

## splicing_config

<pre>
splicing_config(<a href="#splicing_config-resolver_version">resolver_version</a>)
</pre>

arious settings used to configure Cargo manifest splicing behavior.

[rv]: https://doc.rust-lang.org/cargo/reference/resolver.html#resolver-versions


**PARAMETERS**


| Name  | Description | Default Value |
| :------------- | :------------- | :------------- |
| <a id="splicing_config-resolver_version"></a>resolver_version |  The [resolver version][rv] to use in generated Cargo manifests. This flag is **only** used when splicing a manifest from direct package definitions. See <code>crates_repository::packages</code>.   |  <code>"1"</code> |

**RETURNS**

str: A json encoded string of the parameters provided


