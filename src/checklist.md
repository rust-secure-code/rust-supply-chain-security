# Checklist

Legend: 🔒 Must have, ⭐️ Should have, 👍 Nice to have, ℹ️ Info

## Manage dependencies

- [ ] 🔒 Transitive dependencies number is minimized by selecting only the required features ([DEPS-FEATURES](dependencies.html#DEPS-FEATURES))
- [ ] 🔒 Dependencies are vetted ([DEPS-ASSESS](dependencies.html#DEPS-ASSESS))
- [ ] 👍 Dependencies are reviewed using `cargo vet` ([DEPS-VET](dependencies.html#DEPS-VET))
- [ ] 🔒 Known vulnerabilities and unmaintained crates are regularly checked ([DEPS-VULNS](dependencies.html#DEPS-VULNS))
- [ ] 🔒 Dependencies are kept up-to-date ([DEPS-UPDATES](dependencies.html#DEPS-UPDATES))

## Maintain a crate

- [ ] Sources of binary programs include necessary information to make the builds consistent
    - [ ] 🔒 The `Cargo.lock` file is committed to your repository _(when developing binary programs)_ ([MAINTAIN-LOCK](maintain.html#MAINTAIN-LOCK))
    - [ ] 👍 The `rust-toolchain.toml` file is committed to your repository _(when developing binary programs)_ ([MAINTAIN-TOOLCHAIN](maintain.html#MAINTAIN-TOOLCHAIN))
- [ ] 🔒 Crate features allow to only include transitive dependencies required for a given use case ([MAINTAIN-FEATURES](maintain.html#MAINTAIN-FEATURES))
- [ ] Maintaining a `-sys` crate building C/C++ code
    - [ ] ⭐️ Provide flags to control the behavior (MAINTAIN-SRCFLAG) { #MAINTAIN-SRCFLAG }
    - [ ] ⭐️ Use a dedicated `-src` crate (MAINTAIN-SRCCRATE) { #MAINTAIN-SRCCRATE }
- [ ] 🔒 Known vulnerabilities are reported to RustSec ([MAINTAIN-VULNS](maintain.html#MAINTAIN-VULNS))
- [ ] Publication access to crates.io is secured
  - [ ] 🔒 The GitHub accounts owning the crates have 2FA enabled ([MAINTAIN-CRATESIO2FA](maintain.html#MAINTAIN-CRATESIO2FA))
  - [ ] 🔒 All tokens have limited rights ([MAINTAIN-CRATESIOTKN](maintain.html#MAINTAIN-CRATESIOTKN))
- [ ] ⭐️ Apply code security best practices ([MAINTAIN-CODESEC](maintain.html#MAINTAIN-CODESEC))

## Build for production

- [ ] 👍 Build uses a trusted toolchain ([BUILD-TOOLCHAIN](build.html#BUILD-TOOLCHAIN))
- [ ] 🔒 Build uses `--locked` for reproducibility ([BUILD-LOCKED](build.html#BUILD-TOOLCHAIN))
- [ ] 👍 Build is fully reproducible ([BUILD-REPRODUCIBLE](build.html#BUILD-REPRODUCIBLE))
- [ ] ⭐️ Build environment is ephemeral and isolated ([BUILD-ISOLATED](build.html#BUILD-ISOLATED))
- [ ] ⭐️ Build uses `cargo auditable` to embed dependencies information ([BUILD-AUDITABLE](build.html#BUILD-AUDITABLE))
- [ ] 👍 Built binaries are signed ([BUILD-SIGN](build.html#BUILD-SIGN))
- [ ] ℹ️ Build produces a _Software Bill of Materials_ ([BUILD-SBOM](build.html#BUILD-SBOM))

## Run in production

- [ ] ⭐️ Production artifacts are regularly checked for vulnerabilities ([PROD-AUDIT](run.html#PROD-AUDIT))
