# Introduction

## What is _Software Supply Chain Security_?

The _Software Supply Chain_ has been named as an analogy with the "physical"
[supply chain](https://en.wikipedia.org/wiki/Supply_chain),
that goes from raw materials to end consumers, through a system made of multiple links
(factories, delivery, etc.).

> A software supply chain is composed of the components, libraries, tools,
and processes used to develop, build, and publish a software artifact.[^wiki]

In the Rust context, it means more specifically the chain from the sources,
the dependencies (whether on crates.io or not),
build toolchain (`rustc`, `cargo`), CI/CD platforms, binary distribution, etc.

The software supply chain has been an increasingly frequent target
in recent years, as it allows attacking well-secured systems
by focusing on weaker links in their supply chain,
or attacking multiple targets with a single entry point.

Diagram of the software supply chain risks (by the SLSA project)[^threats]:

![The supply chains threats, SLSA project](images/supply-chain-threats.svg)

## About this guide

This guide is not a general introduction to the software supply chain security topic.
To learn more about it, you can
have a look at introductions by vendors like [Red Hat](https://www.redhat.com/en/topics/security/what-is-software-supply-chain-security)
or [snyk](https://snyk.io/series/software-supply-chain-security/),
or the [OpenSSF](https://openssf.org/) and [SLSA](https://slsa.dev)
projects.

This document is aimed at Rust developers, whether working on libraries in the ecosystem or on programs
for end-users (and some parts of the guide are only applicable to one of these situations).
It focuses on currently available tooling for the Rust ecosystem and provides actionable items
whenever possible.

The [checklist](./checklist.md) at the end gives an overview of the recommendations from the whole guide
and helps to prioritize the different topics.

[^wiki]: [_Software Supply Chain_](https://en.wikipedia.org/w/index.php?title=Software_supply_chain&oldid=1148357822#cite_note-1) on Wikipedia (en)
(under _Creative Commons Attribution-ShareAlike License 3.0_)

[^threats]: _Supply chain Levels for Software Artifacts_,
[Supply chain threats](https://slsa.dev/spec/v1.0/threats)
(under _Community Specification License 1.0_)
