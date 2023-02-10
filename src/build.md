# Build for production

The build step is sometimes overlooked when it comes to software security, but is critical,
as recent events like the [SUNSPOT attack](https://www.crowdstrike.com/blog/sunspot-malware-technical-analysis/) on SolarWinds build platform
showed.
Attacks can consist in tricking the build system into building from modified sources 
or inserting malicious software in the build environment.

Another important aspect is to allow users to get visibility over how the program was built,
for example, by providing a precise list of included components, and ideally to allow the users
the check that the produced artifacts actually match this information.

## Build process

### Build toolchain (BUILD-TOOLCHAIN) { #BUILD-TOOLCHAIN }

`rustup` is a very convenient tool for managing Rust toolchains, but [does not yet](https://rust-lang.github.io/rustup/security.html) validate
signatures for the downloaded files. Depending on your requirements, it may be preferable to choose another installation method like
[standalone installers](https://forge.rust-lang.org/infra/other-installation-methods.html#standalone-installers)
which allow you to validate signatures for downloaded files (or a Rust compiler provided by your Linux distribution).

### Locked dependencies (BUILD-LOCKED) { #BUILD-LOCKED }

When building for production, you should have a `Cargo.lock` file in the sources,
and ensure the build commands include the `--locked` flag.

This ensures dependencies' integrity (at least compared to when the lock file was committed), as
the `Cargo.lock` file contains a hash for each crate source, that is checked at build.
It also ensures the dependencies used for production build match what the source repository contains
(git tag, etc.).
If there is an inconsistency between the `Cargo.lock`
and `Cargo.toml` and the `--locked` flag is passed the build will fail
(while the lock file would be automatically updated without it).

### Reproducible builds (BUILD-REPRODUCIBLE) { #BUILD-REPRODUCIBLE }

[Reproducible builds](https://reproducible-builds.org/) designates an effort to make
software re-buildable from sources with the exact same output, allowing anyone to check 
that the provided binaries are actually the product of their sources.
This requires the build process to be deterministic.

It is currently possible to make reproducible builds in Rust, but it requires
some actions to remove build-time absolute paths from the binaries
(which are used in debug and panic messages).
This also leaks the absolute path of the files on the build system which may not be desirable.
To remove them, you need to pass the
[`--remap-path-prefix`](https://doc.rust-lang.org/rustc/command-line-arguments.html#--remap-path-prefix-remap-source-names-in-output)
option to `rustc`.
When using `cargo`, you can use the `RUSTFLAGS` environment variable, with something like:

```shell
RUSTFLAGS="--remap-path-prefix /path/to/build=/static-placeholder" cargo build --release
```

Or use a [`config.toml` file for cargo](https://doc.rust-lang.org/cargo/reference/config.html) with:

```toml
[build]
rustflags = ["--remap-path-prefix /path/to/build=/static-placeholder"]
```

You can pass the option multiple times (and, when multiple re-mappings are given and several
of them match, the last matching one is applied).
This process will become more straightforward once the [RFC 3127](https://rust-lang.github.io/rfcs/3127-trim-paths.html)
gets [implemented](https://github.com/rust-lang/cargo/issues/12137) in cargo.

### Sandboxed builds (BUILD-ISOLATED) { #BUILD-ISOLATED }

As building Rust code runs arbitrary code locally (procedural macros and `build.rs`),
the build environment should be protected against attacks that could lead to persisting malicious software in
the build environment.
To do so, the build environment should be ephemeral and isolated (in a dedicated container, or -even better- a VM).

## Embed dependency list in binaries (BUILD-AUDITABLE) { #BUILD-AUDITABLE }

For traceability, it is useful to be able to get the list of dependencies from a build binary.
This allows, for example, auditing for known vulnerability directly on the binary files.
This is not done by default in Rust builds,
but you can use [`cargo auditable`](https://github.com/rust-secure-code/cargo-auditable) to include this information.

```shell
cargo auditable build --release
```

You can then see the embedded data with [`rust-audit-info`](https://github.com/rust-secure-code/cargo-auditable/tree/master/rust-audit-info):

```shell
$ rust-audit-info my_program | jq
{
  "packages": [
    {
      "name": "aho-corasick",
      "version": "0.7.20",
      "source": "registry",
      "dependencies": [
        83
      ],
      "features": [
        "default",
        "std"
      ]
    }
    # [...]
}
```

**Warning**: Due to [the way some `-sys` crate work](https://internals.rust-lang.org/t/statically-linked-c-c-libraries/17175), some C/C++ native
library may be included in the binary but not visible in the embedded data.

## Signature (BUILD-SIGN) { #BUILD-SIGN }

There is no specific tooling for signing Rust binaires at the moment, but common approaches can apply
You can, for example, use a GPG-based signature.

There is also an open [pre-RFC](https://github.com/trustification/rust-rfcs/blob/sigstore-rfc/text/0000-sigstore-integration.md)
proposing to use [sigstore](https://www.sigstore.dev/) for signing and verifying crates ([topic on IRLO](https://internals.rust-lang.org/t/pre-rfc-using-sigstore-for-signing-and-verifying-crates/18115/31)).

## Software Bill Of Materials (a.k.a. SBOM) (BUILD-SBOM) { #BUILD-SBOM }

[Software Bills Of Materials](https://www.aquasec.com/cloud-native-academy/supply-chain-security/sbom/) for Rust programs usually need
to be created from the sources (as the binaries don't contain enough information).

**Warning**: Due to [the way some `-sys` crate work](https://internals.rust-lang.org/t/statically-linked-c-c-libraries/17175), some C/C++ native
library may be included in the binary but not visible in the
SBOM.

There are available tooling for the two most common SBOM formats in the open-source ecosystems,
[SPDX](https://spdx.dev/) and [CycloneDX](https://cyclonedx.org/).
Choosing a SBOM format is up to you (they should be somehow interoperable).

### CycloneDX with `cargo-cyclonedx`

There is an official CycloneDX cargo subcommand to generate a SBOM
from a Rust project source, [`cargo-cyclonedx`](https://crates.io/crates/cargo-cyclonedx)

```bash
# Produced a `bom.cdx.json` files
cargo cyclonedx --all --format json --output-cdx
```

It understands cargo workspaces and produces a JSON (or XML) file in the current directory.
To aggregate several files, validate or sign them, you can use the [`cyclonedx-cli`](https://github.com/CycloneDX/cyclonedx-cli).

```bash
cyclonedx merge --input-files *.json --output-file myprogram.cdx.json
```

### SPDX and CycloneDX with `cargo-sbom`

There is a cargo subcommand available to generate an SPDX or CycloneDX SBOM from a Rust project source,
[`cargo-sbom`](https://github.com/psastras/sbom-rs).

```bash
# SPDX output (default)
cargo sbom > sbom.spdx.json
# CycloneDX output
cargo sbom --output-format cyclone_dx_json_1_4 > sbom.cdx.json
```
