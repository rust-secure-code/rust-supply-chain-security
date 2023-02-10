# Manage dependencies

`cargo` makes it really easy to use dependencies, so the programs in the Rust ecosystem tend
to use a lot of them.
Despite the obvious advantages for the developers, this also comes with
specific risks.
Let's start by describing them, either direct or somewhere in the dependency tree.

The most common risk is the presence of vulnerabilities in one of the dependencies,
like [Log4Shell](https://en.wikipedia.org/wiki/Log4Shell) in the Java `log4j` logging library,
containing an exploitable remote code execution.

The other main risk is an intentional attack introducing malicious behavior (advertising, attacking developer systems,
compromising[dependencies.md](dependencies.md) build systems, introducing backdoor in final products, etc.) in the 
dependency tree.
There are several possible kinds of malicious crates:

* New crates which look like legitimate ones.
Frequently using a name close to a widely used one (called typosquatting).

  * Such an incident already happened with the [rustdecimal](https://blog.rust-lang.org/2022/05/10/malicious-crate-rustdecimal.html)
    crate, masquerading as the legitimate `rust_decimal` crate.
    It contained a payload targeting Gitlab CI.

* New versions of legitimate crates, either by stealing credentials or
  gaining publication access.

For now the Rust ecosystem has been relatively preserved for these attacks, but it's likely
linked to the lower visibility and popularity (compared to `npm.org` for example),
and not to intrinsic differences in the tooling or usage.

## Choose dependencies

### Minimize dependencies (DEPS-FEATURES) { #DEPS-FEATURES }

The safest dependency is the one you don't include.
Be careful though, the number of dependencies is not always a good measurement of the attack surface
(several crates maintained by the same team don't really multiply the risk).
To get an overview of people and organizations with publish access to your dependencies,
you can use the [cargo supply-chain](https://github.com/rust-secure-code/cargo-supply-chain) tool:

```shell
The following individuals can publish updates for your dependencies:

 1. alexcrichton via crates: bitflags, [...], web-sys
[...]
 139. zrzka via crates: anes

Note: there may be outstanding publisher invitations. crates.io provides no way to list them.
See https://github.com/rust-lang/crates.io/issues/2868 for more info.

All members of the following teams can publish updates for your dependencies:

 1. "github:rust-bus:maintainers" (https://github.com/rust-bus) via crates: [...]
[...]
 35. "github:tower-rs:publish" (https://github.com/tower-rs) via crates: tower-service

Github teams are black boxes. It's impossible to get the member list without explicit permission.
```

In order to minimize the number of dependencies, and your attack surface, you can:

* Explore your dependency tree with `cargo tree`/`cargo supply-chain` to see if you spot something to remove
* Minimizing the dependency features.
  Crates tend to include a lot of features by default for convenience,
* but you can usually remove a part of the dependencies by disabling default features and only selecting the ones you actually need.

```toml
mylibrary = { version = "1", default-features = false, features = [ "required-feature" ] }
```

### Assess a dependency (DEPS-ASSESS) { #DEPS-ASSESS }

There is a [very good checklist](https://gist.github.com/repi/d98bf9c202ec567fd67ef9e31152f43f)
covering many aspects of crate evaluation published (and used internally) by [Embark](https://www.embark-studios.com/).

When considering adding a dependency, there are a few things
you can do to lower the different risks.
Auditing the code is always an option, but is not scalable, especially including all future version upgrades.
There are several initiatives intending to share the review
work in the community,
like [`cargo-crev`](https://github.com/crev-dev/cargo-crev) and [`cargo-vet`](https://mozilla.github.io/cargo-vet/).

Note: When reviewing a crate, don't use the repository as a reference as it may diverge
from the content uploaded to crates.io.
Instead, you can download the crate (using URLs like: https://crates.io/api/v1/crates/serde/1.0.0/download),
use the source view on docs.rs (or an external service like [Sourcegraph](https://sourcegraph.com/search)
([example](https://sourcegraph.com/crates/regex@v1.1.0/-/blob/src/compile.rs)),
used by `cargo vet` for reviews, which is more convenient for comparing versions).

Note: Be careful when opening untrusted crates in your editor/IDE,
as even only opening it can run arbitrary code in procedural macros.
Use a simple text editor (without LSP-like features) or make use of
features like the [restricted mode](https://code.visualstudio.com/docs/editor/workspace-trust) in VSCode.

To prevent potential vulnerabilities in memory management,
a focus on `unsafe` code is often needed in code reviews.
Tooling like [`cargo-geiger`](https://github.com/rust-secure-code/cargo-geiger) can help
track the use of unsafe code in dependencies.

### Assess dependencies with `cargo vet` (DEPS-VET) { #DEPS-VET }

`cargo vet` is a project started by Mozilla
(and now also [used by Google](https://opensource.googleblog.com/2023/05/open-sourcing-our-rust-crate-audits.html))
to formalize crate audits in an organization, and be able to share
these with the community.

As this topic is all a matter of trust, by default `cargo vet` works with local audits
you make yourselves.
You can also [import audit results](https://mozilla.github.io/cargo-vet/importing-audits.html)
from selected sources you trust, in the form of a git repository
URL. There is no centralized data source nor default trusted source.

```toml
[imports.mozilla]
url = "https://raw.githubusercontent.com/mozilla/supply-chain/main/audits.toml"

[imports.google]
url = "https://raw.githubusercontent.com/google/supply-chain/main/audits.toml"
```

You can also [trust specific publishers](https://mozilla.github.io/cargo-vet/trusting-publishers.html).
This lowers the level of scrutiny (as it does not protect against some threats)
but allows focusing on more risky packages.

## Maintain dependencies over time

Once you have chosen to use a dependency, you need to keep track of it
over time.

### Vulnerability and maintenance check (DEPS-VULNS) { #DEPS-VULNS }

You should also monitor for vulnerabilities and unmaintained crates using `cargo audit` or `cargo deny` regularly
(e.g. with a daily CI job).
[GitHub's tooling](https://github.blog/2022-06-06-github-brings-supply-chain-security-features-to-the-rust-community/)
can also help
track security vulnerabilities in your project.

### Regular updates (DEPS-UPDATES) { #DEPS-UPDATES }

You should also try to stay up to date; this helps get bug fixes and improvements,
and also avoids using unmaintained versions,
ensuring to get easy access to security fixes in case of security vulnerability.
Use [`cargo outdated`](https://github.com/kbknapp/cargo-outdated) to show possible updates.
