# ts-grammar-action

A GitHub Action for installing tree-sitter and language grammars via [tsdl](https://github.com/stackmystack/tsdl).

This action wraps the official [`tree-sitter/setup-action`](https://github.com/tree-sitter/setup-action) and uses `tsdl` to build and install language-specific grammar shared libraries.

## Features

  - 🌲 Uses official `tree-sitter/setup-action` for library and CLI installation
  - 📦 Builds grammar shared libraries (`.so` files) via [tsdl](https://github.com/stackmystack/tsdl)
  - 📌 Pinned grammar versions by default (override with `grammar-*-ref` inputs)
  - ⚡ Caching support via the official action
  - 🔧 Configures environment variables (`TREE_SITTER_*_PATH`) for each grammar
  - 🐧 Linux and macOS support
## Supported Grammars

| Grammar | Input | Repository |
| --- | --- | --- |
| Bash | `grammar-bash` | [tree-sitter/tree-sitter-bash](https://github.com/tree-sitter/tree-sitter-bash) |
| JSON | `grammar-json` | [tree-sitter/tree-sitter-json](https://github.com/tree-sitter/tree-sitter-json) |
| TOML | `grammar-toml` | [tree-sitter-grammars/tree-sitter-toml](https://github.com/tree-sitter-grammars/tree-sitter-toml) |
| RBS | `grammar-rbs` | [joker1007/tree-sitter-rbs](https://github.com/joker1007/tree-sitter-rbs) |

> **Note**: JSONC (JSON with Comments) is supported by the main `tree-sitter-json` grammar as of v0.24.0. A separate jsonc grammar is no longer needed.

## Usage

### Version Reference

Use one of the following version references:
  - `@v2` - Latest stable v2.x release (recommended for production)
  - `@v2.0.0` - Specific version
  - `@main` - Latest development version (may be unstable)
### Basic Usage

Install tree-sitter library and selected grammars:

``` yaml
steps:
  - uses: actions/checkout@v4

  - name: Install tree-sitter with grammars
    uses: kettle-rb/ts-grammar-action@v2
    with:
      grammar-bash: true
      grammar-toml: true
```

### All Options

``` yaml
steps:
  - uses: actions/checkout@v4

  - name: Install tree-sitter with grammars
    uses: kettle-rb/ts-grammar-action@v2
    with:
      # Tree-sitter library/CLI options
      install-cli: false          # Install tree-sitter CLI (default: false)
      install-lib: true           # Install libtree-sitter (default: true)
      tree-sitter-ref: latest     # tree-sitter version (default: latest)

      # Rust toolchain (for tree_stump gem or CLI build)
      setup-rust: false           # Install Rust toolchain (default: false)
      rust-toolchain: stable      # Rust version (default: stable)

      # Java/jtreesitter (for JRuby Java backend)
      # Note: setup-jtreesitter: true automatically installs Java JDK
      setup-java: false           # Install Java JDK only (default: false)
      java-version: "23"          # Java version (default: 23)
      java-distribution: temurin  # Java distribution (default: temurin)
      setup-jtreesitter: false    # Download jtreesitter JAR + install Java (default: false)
      jtreesitter-version: "0.24.0"  # jtreesitter version (default: 0.24.0)
      jtreesitter-install-dir: /usr/local/share/java  # JAR install directory

      # tsdl options
      tsdl-version: v2.0.0       # tsdl version (default: v2.0.0)

      # Grammar selections (all default to false)
      grammar-bash: false
      grammar-bash-ref: v0.25.1      # Pin grammar version
      grammar-json: false
      grammar-json-ref: v0.24.8
      grammar-toml: false
      grammar-toml-ref: v0.7.0
      grammar-rbs: false
      grammar-rbs-ref: v0.2.2

      # Installation prefix (default: /usr/local)
      grammar-install-prefix: /usr/local
```

### Using with tree\_stump (Rust Backend)

If you're using the [tree\_stump](https://github.com/joker1007/tree_stump) gem for tree\_haver's Rust backend on MRI Ruby, you need Rust installed to compile it:

``` yaml
steps:
  - uses: actions/checkout@v4

  - name: Install tree-sitter with Rust and grammars
    uses: kettle-rb/ts-grammar-action@v2
    with:
      setup-rust: true            # Install Rust toolchain
      rust-toolchain: stable      # Use stable Rust
      grammar-toml: true
      grammar-bash: true

  - name: Setup Ruby
    uses: ruby/setup-ruby@v1
    with:
      ruby-version: '3.3'
      bundler-cache: true

  - name: Run tests
    run: bundle exec rspec
```

### Using with jtreesitter (JRuby Java Backend)

For JRuby workflows using tree\_haver's Java backend with [jtreesitter](https://github.com/tree-sitter/java-tree-sitter):

> **Note**: Setting `setup-jtreesitter: true` automatically installs Java JDK - you don't need to also set `setup-java: true`.

``` yaml
steps:
  - uses: actions/checkout@v4

  - name: Install tree-sitter with Java and grammars
    uses: kettle-rb/ts-grammar-action@v2
    with:
      setup-jtreesitter: true     # Automatically installs Java JDK + jtreesitter JAR
      java-version: "23"          # Java version to install
      grammar-toml: true
      grammar-bash: true

  - name: Setup JRuby
    uses: ruby/setup-ruby@v1
    with:
      ruby-version: jruby
      bundler-cache: true

  - name: Run tests
    run: bundle exec rspec
    env:
      # These are automatically set by the action:
      # TREE_SITTER_JAVA_JARS_DIR: /usr/local/share/java
      # CLASSPATH: /usr/local/share/java/jtreesitter-0.24.0.jar
```

### Using with Ruby Gems

This action is designed to work with the [tree\_haver](https://github.com/kettle-rb/tree_haver) gem and the `*-merge` gem family for AST-based file merging:

``` yaml
steps:
  - uses: actions/checkout@v4

  - name: Install tree-sitter and grammars
    uses: kettle-rb/ts-grammar-action@v2
    with:
      grammar-toml: true
      grammar-bash: true
      grammar-json: true

  - name: Setup Ruby
    uses: ruby/setup-ruby@v1
    with:
      ruby-version: '3.3'
      bundler-cache: true

  - name: Run tests
    run: bundle exec rspec
    env:
      # These are automatically set by the action:
      # TREE_SITTER_TOML_PATH: /usr/local/lib/libtree-sitter-toml.so
      # TREE_SITTER_BASH_PATH: /usr/local/lib/libtree-sitter-bash.so
      # TREE_SITTER_JSON_PATH: /usr/local/lib/libtree-sitter-json.so
```

## Outputs

| Output | Description |
| --- | --- |
| `grammars-installed` | Comma-separated list of installed grammar names |
| `lib-path` | Path to installed grammar libraries |
| `tsdl-version` | Installed tsdl version |
| `rust-installed` | Whether Rust toolchain was installed (`true`/`false`) |
| `java-installed` | Whether Java JDK was installed (`true`/`false`) |
| `jtreesitter-installed` | Whether jtreesitter JAR was installed (`true`/`false`) |
| `jtreesitter-jar-path` | Path to jtreesitter JAR file |

### Example: Using Outputs

``` yaml
steps:
  - uses: actions/checkout@v4

  - name: Install grammars
    id: grammars
    uses: kettle-rb/ts-grammar-action@v2
    with:
      grammar-toml: true
      grammar-json: true

  - name: Show installed grammars
    run: |
      echo "Installed: ${{ steps.grammars.outputs.grammars-installed }}"
      echo "Library path: ${{ steps.grammars.outputs.lib-path }}"
      ls -la ${{ steps.grammars.outputs.lib-path }}/libtree-sitter-*.so
```

## Environment Variables

The action automatically sets environment variables for each installed component:

### Grammar Environment Variables

| Variable | Example Value |
| --- | --- |
| `TREE_SITTER_BASH_PATH` | `/usr/local/lib/libtree-sitter-bash.so` |
| `TREE_SITTER_JSON_PATH` | `/usr/local/lib/libtree-sitter-json.so` |
| `TREE_SITTER_TOML_PATH` | `/usr/local/lib/libtree-sitter-toml.so` |
| `TREE_SITTER_RBS_PATH` | `/usr/local/lib/libtree-sitter-rbs.so` |
| `LD_LIBRARY_PATH` | Updated to include grammar library directory |

### Java/jtreesitter Environment Variables

| Variable | Example Value |
| --- | --- |
| `TREE_SITTER_JAVA_JARS_DIR` | `/usr/local/share/java` |
| `CLASSPATH` | `/usr/local/share/java/jtreesitter-0.24.0.jar` |
| `JAVA_HOME` | Set by `actions/setup-java` |

These environment variables are used by [tree\_haver](https://github.com/kettle-rb/tree_haver) to locate grammar libraries and Java JARs.

## Requirements

  - **Linux or macOS** - Windows support may be added in the future
  - **sudo** - For installing to system directories
## How It Works

1.  **Java Setup** (optional): Installs Java JDK via `actions/setup-java` if `setup-java` or `setup-jtreesitter` is enabled
2.  **jtreesitter Setup** (optional): Downloads jtreesitter JAR from Maven Central for JRuby's Java backend
3.  **Rust Setup** (optional): Installs Rust toolchain via `dtolnay/rust-toolchain` if `setup-rust` is enabled
4.  **Tree-sitter Setup**: Uses the official `tree-sitter/setup-action` to install the tree-sitter library and optionally the CLI
5.  **tsdl Install**: Downloads the [tsdl](https://github.com/stackmystack/tsdl) binary from GitHub releases
6.  **Grammar Build**: Generates a `parsers.toml` config from inputs and runs `tsdl build` to compile grammars
7.  **Configuration**: Sets up environment variables for grammar and JAR discovery
## Comparison with tree-sitter/setup-action

| Feature | tree-sitter/setup-action | ts-grammar-action |
| --- | --- | --- |
| Library installation | ✅ | ✅ (via delegation) |
| CLI installation | ✅ | ✅ (via delegation) |
| Caching | ✅ | ✅ (via delegation) |
| Grammar installation | ❌ | ✅ (via [tsdl](https://github.com/stackmystack/tsdl)) |
| Pinned grammar versions | ❌ | ✅ |
| Rust toolchain | ✅ (for CLI only) | ✅ (for tree\_stump gem) |
| Java JDK | ❌ | ✅ |
| jtreesitter JAR | ❌ | ✅ |
| Environment variables | Library paths only | Library + grammar + Java |

## Related Projects

This project is part of the [kettle-rb](https://github.com/kettle-rb) family of tools.
This action specifically facilitates testing many of the gems in the merge gem family in GitHub Actions.

### The `*-merge` Gem Family

The `*-merge` gem family provides intelligent, AST-based merging for various file formats. At the foundation is [tree_haver][tree_haver], which provides a unified cross-Ruby parsing API that works seamlessly across MRI, JRuby, and TruffleRuby.

| Gem                                      | Language<br>/ Format | Parser Backend(s)                                                                                   | Description                                                                      |
|------------------------------------------|----------------------|-----------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| [tree_haver][tree_haver]                 | Multi                | MRI C, Rust, FFI, Java, Prism, Psych, Commonmarker, Markly, Citrus                                  | **Foundation**: Cross-Ruby adapter for parsing libraries (like Faraday for HTTP) |
| [ast-merge][ast-merge]                   | Text                 | internal                                                                                            | **Infrastructure**: Shared base classes and merge logic for all `*-merge` gems   |
| [bash-merge][bash-merge]                 | Bash                 | [tree-sitter-bash][ts-bash] (via tree_haver)                                                        | Smart merge for Bash scripts                                                     |
| [commonmarker-merge][commonmarker-merge] | Markdown             | [Commonmarker][commonmarker] (via tree_haver)                                                       | Smart merge for Markdown (CommonMark via comrak Rust)                            |
| [dotenv-merge][dotenv-merge]             | Dotenv               | internal                                                                                            | Smart merge for `.env` files                                                     |
| [json-merge][json-merge]                 | JSON                 | [tree-sitter-json][ts-json] (via tree_haver)                                                        | Smart merge for JSON files                                                       |
| [jsonc-merge][jsonc-merge]               | JSONC                | [tree-sitter-json][ts-json] (via tree_haver)                                                        | Smart merge for JSON with Comments                                               |
| [markdown-merge][markdown-merge]         | Markdown             | [Commonmarker][commonmarker] / [Markly][markly] (via tree_haver)                                    | **Foundation**: Shared base for Markdown mergers with inner code block merging   |
| [markly-merge][markly-merge]             | Markdown             | [Markly][markly] (via tree_haver)                                                                   | Smart merge for Markdown (CommonMark via cmark-gfm C)                            |
| [prism-merge][prism-merge]               | Ruby                 | [Prism][prism] (`prism` std lib gem)                                                                | Smart merge for Ruby source files                                                |
| [psych-merge][psych-merge]               | YAML                 | [Psych][psych] (`psych` std lib gem)                                                                | Smart merge for YAML files                                                       |
| [rbs-merge][rbs-merge]                   | RBS                  | [tree-sitter-rbs][ts-rbs] (via tree_haver), [RBS][rbs] (`rbs` std lib gem)                          | Smart merge for Ruby type signatures                                             |
| [toml-merge][toml-merge]                 | TOML                 | [Citrus + toml-rb][toml-rb] (default, via tree_haver), [tree-sitter-toml][ts-toml] (via tree_haver) | Smart merge for TOML files                                                       |

#### Backend Platform Compatibility

tree_haver supports multiple parsing backends, but not all backends work on all Ruby platforms:

| Platform 👉️<br> TreeHaver Backend 👇️         | MRI | JRuby | TruffleRuby | Notes                                               |
|------------------------------------------------|:---:|:-----:|:-----------:|-----------------------------------------------------|
| **MRI** ([ruby_tree_sitter][ruby_tree_sitter]) |  ✅  |   ❌   |      ❌      | C extension, MRI only                               |
| **Rust** ([tree_stump][tree_stump])            |  ✅  |   ❌   |      ❌      | Rust extension via magnus/rb-sys, MRI only          |
| **FFI**                                        |  ✅  |   ✅   |      ❌      | TruffleRuby's FFI doesn't support `STRUCT_BY_VALUE` |
| **Java** ([jtreesitter][jtreesitter])          |  ❌  |   ✅   |      ❌      | JRuby only, requires grammar JARs                   |
| **Prism**                                      |  ✅  |   ✅   |      ✅      | Ruby parsing, stdlib in Ruby 3.4+                   |
| **Psych**                                      |  ✅  |   ✅   |      ✅      | YAML parsing, stdlib                                |
| **Citrus**                                     |  ✅  |   ✅   |      ✅      | Pure Ruby, no native dependencies                   |
| **Commonmarker**                               |  ✅  |   ❌   |      ❓      | Rust extension for Markdown                         |
| **Markly**                                     |  ✅  |   ❌   |      ❓      | C extension for Markdown                            |

**Legend**: ✅ = Works, ❌ = Does not work, ❓ = Untested

**Why some backends don't work on certain platforms**:

- **JRuby**: Runs on the JVM; cannot load native C/Rust extensions (`.so` files)
- **TruffleRuby**: Has C API emulation via Sulong/LLVM, but it doesn't expose all MRI internals that native extensions require (e.g., `RBasic.flags`, `rb_gc_writebarrier`)
- **FFI on TruffleRuby**: TruffleRuby's FFI implementation doesn't support returning structs by value, which tree-sitter's C API requires

**Example implementations** for the gem templating use case:

| Gem                      | Purpose         | Description                                   |
|--------------------------|-----------------|-----------------------------------------------|
| [kettle-dev][kettle-dev] | Gem Development | Gem templating tool using `*-merge` gems      |
| [kettle-jem][kettle-jem] | Gem Templating  | Gem template library with smart merge support |

[tree_haver]: https://github.com/kettle-rb/tree_haver
[ast-merge]: https://github.com/kettle-rb/ast-merge
[prism-merge]: https://github.com/kettle-rb/prism-merge
[psych-merge]: https://github.com/kettle-rb/psych-merge
[json-merge]: https://github.com/kettle-rb/json-merge
[jsonc-merge]: https://github.com/kettle-rb/jsonc-merge
[bash-merge]: https://github.com/kettle-rb/bash-merge
[rbs-merge]: https://github.com/kettle-rb/rbs-merge
[dotenv-merge]: https://github.com/kettle-rb/dotenv-merge
[toml-merge]: https://github.com/kettle-rb/toml-merge
[markdown-merge]: https://github.com/kettle-rb/markdown-merge
[markly-merge]: https://github.com/kettle-rb/markly-merge
[commonmarker-merge]: https://github.com/kettle-rb/commonmarker-merge
[kettle-dev]: https://github.com/kettle-rb/kettle-dev
[kettle-jem]: https://github.com/kettle-rb/kettle-jem
[prism]: https://github.com/ruby/prism
[psych]: https://github.com/ruby/psych
[ts-json]: https://github.com/tree-sitter/tree-sitter-json
[ts-bash]: https://github.com/tree-sitter/tree-sitter-bash
[ts-rbs]: https://github.com/joker1007/tree-sitter-rbs
[ts-toml]: https://github.com/tree-sitter-grammars/tree-sitter-toml
[dotenv]: https://github.com/bkeepers/dotenv
[rbs]: https://github.com/ruby/rbs
[toml-rb]: https://github.com/emancu/toml-rb
[markly]: https://github.com/ioquatix/markly
[commonmarker]: https://github.com/gjtorikian/commonmarker
[ruby_tree_sitter]: https://github.com/Faveod/ruby-tree-sitter
[tree_stump]: https://github.com/joker1007/tree_stump
[jtreesitter]: https://central.sonatype.com/artifact/io.github.tree-sitter/jtreesitter

## 🦷 FLOSS Funding

While kettle-rb tools are free software and will always be, the project would benefit immensely from some funding.
Raising a monthly budget of... "dollars" would make the project more sustainable.

We welcome both individual and corporate sponsors\! We also offer a
wide array of funding channels to account for your preferences
(although currently [Open Collective](https://opencollective.com/kettle-rb) is our preferred funding platform).

**If you're working in a company that's making significant use of kettle-rb tools we'd
appreciate it if you suggest to your company to become a kettle-rb sponsor.**

You can support the development of kettle-rb tools via
[GitHub Sponsors](https://github.com/sponsors/pboling),
[Liberapay](https://liberapay.com/pboling/donate),
[PayPal](https://www.paypal.com/paypalme/peterboling),
[Open Collective](https://opencollective.com/kettle-rb)
and [Tidelift](https://tidelift.com/subscription/pkg/rubygems-ts-grammar-action?utm_source=rubygems-ts-grammar-action&utm_medium=referral&utm_campaign=readme).

| 📍 NOTE |
| --- |
| If doing a sponsorship in the form of donation is problematic for your company <br/> from an accounting standpoint, we'd recommend the use of Tidelift, <br/> where you can get a support-like subscription instead. |

### Open Collective for Individuals

Support us with a monthly donation and help us continue our activities. \[[Become a backer](https://opencollective.com/kettle-rb#backer)\]

NOTE: [kettle-readme-backers](https://github.com/kettle-rb/ts-grammar-action/blob/main/exe/kettle-readme-backers) updates this list every day, automatically.

<!-- OPENCOLLECTIVE-INDIVIDUALS:START -->
No backers yet. Be the first\!
<!-- OPENCOLLECTIVE-INDIVIDUALS:END -->

### Open Collective for Organizations

Become a sponsor and get your logo on our README on GitHub with a link to your site. \[[Become a sponsor](https://opencollective.com/kettle-rb#sponsor)\]

NOTE: [kettle-readme-backers](https://github.com/kettle-rb/ts-grammar-action/blob/main/exe/kettle-readme-backers) updates this list every day, automatically.

<!-- OPENCOLLECTIVE-ORGANIZATIONS:START -->
No sponsors yet. Be the first\!
<!-- OPENCOLLECTIVE-ORGANIZATIONS:END -->

[kettle-readme-backers]: https://github.com/kettle-rb/ts-grammar-action/blob/main/exe/kettle-readme-backers

## 🔐 Security

See [SECURITY.md](SECURITY.md).

## 🤝 Contributing

If you need some ideas of where to help, see [issues](https://github.com/kettle-rb/ts-grammar-action/issues), or [PRs](https://github.com/kettle-rb/ts-grammar-action/pulls),
or use the action and think about how it could be better.

See [CONTRIBUTING.md](CONTRIBUTING.md) for more detailed instructions.

We [![Keep A Changelog](https://img.shields.io/badge/keep--a--changelog-1.0.0-34495e.svg?style=flat)](https://keepachangelog.com/en/1.0.0/) so if you make changes, remember to update it.

### 🚀 Release Instructions

See [CONTRIBUTING.md](CONTRIBUTING.md).

### 🪇 Code of Conduct

Everyone interacting with this project's codebases, issue trackers,
chat rooms and mailing lists agrees to follow the [![Contributor Covenant 2.1](https://img.shields.io/badge/Contributor_Covenant-2.1-259D6C.svg)](CODE_OF_CONDUCT.md).

## 🌈 Contributors

[![Contributors](https://contrib.rocks/image?repo=kettle-rb/ts-grammar-action)](https://github.com/kettle-rb/ts-grammar-action/graphs/contributors)

Made with [contributors-img](https://contrib.rocks).

Also see GitLab Contributors: <https://gitlab.com/kettle-rb/ts-grammar-action/-/graphs/main>

<details>
    <summary>⭐️ Star History</summary>

<a href="https://star-history.com/#kettle-rb/ts-grammar-action&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=kettle-rb/ts-grammar-action&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=kettle-rb/ts-grammar-action&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=kettle-rb/ts-grammar-action&type=Date" />
 </picture>
</a>

</details>

## 📄 License

The gem is available as open source under the terms of
the [MIT License](LICENSE.txt) [![License: MIT](https://img.shields.io/badge/License-MIT-259D6C.svg)](https://opensource.org/licenses/MIT).
See [LICENSE.txt](LICENSE.txt) for the official [Copyright Notice](https://opensource.stackexchange.com/questions/5778/why-do-licenses-such-as-the-mit-license-specify-a-single-year).

### © Copyright

<ul>
    <li>
        Copyright (c) 2025 Peter H. Boling, of
        <a href="https://discord.gg/3qme4XHNKN">
            Galtzo.com
            <picture>
              <img src="https://logos.galtzo.com/assets/images/galtzo-floss/avatar-128px-blank.svg" alt="Galtzo.com Logo (Wordless) by Aboling0, CC BY-SA 4.0" width="24">
            </picture>
        </a>, and ts-grammar-action contributors.
    </li>
</ul>

## 🤑 A request for help

Maintainers have teeth and need to pay their dentists.
After getting laid off in an RIF in March, and encountering difficulty finding a new one,
I began spending most of my time building open source tools.
I'm hoping to be able to pay for my kids' health insurance this month,
so if you value the work I am doing, I need your support.
Please consider sponsoring me or the project.

To join the community or get help 👇️ Join the Discord.

[![Live Chat on Discord](https://img.shields.io/discord/1373797679469170758?style=for-the-badge&logo=discord)](https://discord.gg/3qme4XHNKN)

To say "thanks\!" ☝️ Join the Discord or 👇️ send money.

[![Sponsor kettle-rb/ts-grammar-action on Open Source Collective](https://img.shields.io/opencollective/all/kettle-rb?style=for-the-badge)](https://opencollective.com/kettle-rb) 💌 [![Sponsor me on GitHub Sponsors](https://img.shields.io/badge/Sponsor_Me!-pboling-blue?style=for-the-badge&logo=github)](https://github.com/sponsors/pboling) 💌 [![Sponsor me on Liberapay](https://img.shields.io/liberapay/goal/pboling.svg?style=for-the-badge&logo=liberapay&color=a51611)](https://liberapay.com/pboling/donate) 💌 [![Donate on PayPal](https://img.shields.io/badge/donate-paypal-a51611.svg?style=for-the-badge&logo=paypal&color=0A0A0A)](https://www.paypal.com/paypalme/peterboling)

### Please give the project a star ⭐ ♥.

Thanks for RTFM. ☺️

[⛳liberapay-img]: https://img.shields.io/liberapay/goal/pboling.svg?logo=liberapay&color=a51611&style=flat
[⛳liberapay-bottom-img]: https://img.shields.io/liberapay/goal/pboling.svg?style=for-the-badge&logo=liberapay&color=a51611
[⛳liberapay]: https://liberapay.com/pboling/donate
[🖇osc-all-img]: https://img.shields.io/opencollective/all/kettle-rb
[🖇osc-sponsors-img]: https://img.shields.io/opencollective/sponsors/kettle-rb
[🖇osc-backers-img]: https://img.shields.io/opencollective/backers/kettle-rb
[🖇osc-backers]: https://opencollective.com/kettle-rb#backer
[🖇osc-backers-i]: https://opencollective.com/kettle-rb/backers/badge.svg?style=flat
[🖇osc-sponsors]: https://opencollective.com/kettle-rb#sponsor
[🖇osc-sponsors-i]: https://opencollective.com/kettle-rb/sponsors/badge.svg?style=flat
[🖇osc-all-bottom-img]: https://img.shields.io/opencollective/all/kettle-rb?style=for-the-badge
[🖇osc-sponsors-bottom-img]: https://img.shields.io/opencollective/sponsors/kettle-rb?style=for-the-badge
[🖇osc-backers-bottom-img]: https://img.shields.io/opencollective/backers/kettle-rb?style=for-the-badge
[🖇osc]: https://opencollective.com/kettle-rb
[🖇sponsor-img]: https://img.shields.io/badge/Sponsor_Me!-pboling.svg?style=social&logo=github
[🖇sponsor-bottom-img]: https://img.shields.io/badge/Sponsor_Me!-pboling-blue?style=for-the-badge&logo=github
[🖇sponsor]: https://github.com/sponsors/pboling
[🖇polar-img]: https://img.shields.io/badge/polar-donate-a51611.svg?style=flat
[🖇polar]: https://polar.sh/pboling
[🖇kofi-img]: https://img.shields.io/badge/ko--fi-%E2%9C%93-a51611.svg?style=flat
[🖇kofi]: https://ko-fi.com/O5O86SNP4
[🖇patreon-img]: https://img.shields.io/badge/patreon-donate-a51611.svg?style=flat
[🖇patreon]: https://patreon.com/galtzo
[🖇buyme-small-img]: https://img.shields.io/badge/buy_me_a_coffee-%E2%9C%93-a51611.svg?style=flat
[🖇buyme-img]: https://img.buymeacoffee.com/button-api/?text=Buy%20me%20a%20latte&emoji=&slug=pboling&button_colour=FFDD00&font_colour=000000&font_family=Cookie&outline_colour=000000&coffee_colour=ffffff
[🖇buyme]: https://www.buymeacoffee.com/pboling
[🖇paypal-img]: https://img.shields.io/badge/donate-paypal-a51611.svg?style=flat&logo=paypal
[🖇paypal-bottom-img]: https://img.shields.io/badge/donate-paypal-a51611.svg?style=for-the-badge&logo=paypal&color=0A0A0A
[🖇paypal]: https://www.paypal.com/paypalme/peterboling
[🖇floss-funding.dev]: https://floss-funding.dev
[🖇floss-funding-gem]: https://github.com/galtzo-floss/floss_funding
[✉️discord-invite]: https://discord.gg/3qme4XHNKN
[✉️discord-invite-img-ftb]: https://img.shields.io/discord/1373797679469170758?style=for-the-badge&logo=discord
[✉️ruby-friends-img]: https://img.shields.io/badge/daily.dev-%F0%9F%92%8E_Ruby_Friends-0A0A0A?style=for-the-badge&logo=dailydotdev&logoColor=white
[✉️ruby-friends]: https://app.daily.dev/squads/rubyfriends

[✇bundle-group-pattern]: https://gist.github.com/pboling/4564780
[⛳️tag-img]: https://img.shields.io/github/tag/kettle-rb/ts-grammar-action.svg
[⛳️tag]: http://github.com/kettle-rb/ts-grammar-action/releases
[🚂maint-blog]: http://www.railsbling.com/tags/ts-grammar-action
[🚂maint-blog-img]: https://img.shields.io/badge/blog-railsbling-0093D0.svg?style=for-the-badge&logo=rubyonrails&logoColor=orange
[🚂maint-contact]: http://www.railsbling.com/contact
[🚂maint-contact-img]: https://img.shields.io/badge/Contact-Maintainer-0093D0.svg?style=flat&logo=rubyonrails&logoColor=red
[💖🖇linkedin]: http://www.linkedin.com/in/peterboling
[💖🖇linkedin-img]: https://img.shields.io/badge/PeterBoling-LinkedIn-0B66C2?style=flat&logo=newjapanprowrestling
[💖✌️wellfound]: https://wellfound.com/u/peter-boling
[💖✌️wellfound-img]: https://img.shields.io/badge/peter--boling-orange?style=flat&logo=wellfound
[💖💲crunchbase]: https://www.crunchbase.com/person/peter-boling
[💖💲crunchbase-img]: https://img.shields.io/badge/peter--boling-purple?style=flat&logo=crunchbase
[💖🐘ruby-mast]: https://ruby.social/@galtzo
[💖🐘ruby-mast-img]: https://img.shields.io/mastodon/follow/109447111526622197?domain=https://ruby.social&style=flat&logo=mastodon&label=Ruby%20@galtzo
[💖🦋bluesky]: https://bsky.app/profile/galtzo.com
[💖🦋bluesky-img]: https://img.shields.io/badge/@galtzo.com-0285FF?style=flat&logo=bluesky&logoColor=white
[💖🌳linktree]: https://linktr.ee/galtzo
[💖🌳linktree-img]: https://img.shields.io/badge/galtzo-purple?style=flat&logo=linktree
[💖💁🏼‍♂️devto]: https://dev.to/galtzo
[💖💁🏼‍♂️devto-img]: https://img.shields.io/badge/dev.to-0A0A0A?style=flat&logo=devdotto&logoColor=white
[💖💁🏼‍♂️aboutme]: https://about.me/peter.boling
[💖💁🏼‍♂️aboutme-img]: https://img.shields.io/badge/about.me-0A0A0A?style=flat&logo=aboutme&logoColor=white
[💖🧊berg]: https://codeberg.org/pboling
[💖🐙hub]: https://github.org/pboling
[💖🛖hut]: https://sr.ht/~galtzo/
[💖🧪lab]: https://gitlab.com/pboling
[💁🏼‍♂️peterboling]: http://www.peterboling.com
[🚂railsbling]: http://www.railsbling.com
[📜src-gh-img]: https://img.shields.io/badge/GitHub-238636?style=for-the-badge&logo=Github&logoColor=green
[📜src-gh]: https://github.com/kettle-rb/ts-grammar-action
[📜docs-cr-rd-img]: https://img.shields.io/badge/RubyDoc-Current_Release-943CD2?style=for-the-badge&logo=readthedocs&logoColor=white
[📜docs-head-rd-img]: https://img.shields.io/badge/YARD_on_Galtzo.com-HEAD-943CD2?style=for-the-badge&logo=readthedocs&logoColor=white
[📜gh-wiki]: https://github.com/kettle-rb/ts-grammar-action/wiki
[📜gh-wiki-img]: https://img.shields.io/badge/wiki-examples-943CD2.svg?style=for-the-badge&logo=github&logoColor=white
[👽oss-help]: https://www.codetriage.com/kettle-rb/ts-grammar-action
[👽oss-helpi]: https://www.codetriage.com/kettle-rb/ts-grammar-action/badges/users.svg
[🤝gh-issues]: https://github.com/kettle-rb/ts-grammar-action/issues
[🤝gh-pulls]: https://github.com/kettle-rb/ts-grammar-action/pulls
[🤝contributing]: CONTRIBUTING.md
[🖐contrib-rocks]: https://contrib.rocks
[🖐contributors]: https://github.com/kettle-rb/ts-grammar-action/graphs/contributors
[🖐contributors-img]: https://contrib.rocks/image?repo=kettle-rb/ts-grammar-action
[🪇conduct]: CODE_OF_CONDUCT.md
[🪇conduct-img]: https://img.shields.io/badge/Contributor_Covenant-2.1-259D6C.svg
[📌gitmoji]: https://gitmoji.dev
[📌gitmoji-img]: https://img.shields.io/badge/gitmoji_commits-%20%F0%9F%98%9C%20%F0%9F%98%8D-34495e.svg?style=flat-square
[🔐security]: SECURITY.md
[🔐security-img]: https://img.shields.io/badge/security-policy-259D6C.svg?style=flat
[📄copyright-notice-explainer]: https://opensource.stackexchange.com/questions/5778/why-do-licenses-such-as-the-mit-license-specify-a-single-year
[📄license]: LICENSE.txt
[📄license-ref]: https://opensource.org/licenses/MIT
[📄license-img]: https://img.shields.io/badge/License-MIT-259D6C.svg
[📄license-compat]: https://dev.to/galtzo/how-to-check-license-compatibility-41h0
[📄license-compat-img]: https://img.shields.io/badge/Apache_Compatible:_Category_A-%E2%9C%93-259D6C.svg?style=flat&logo=Apache
[📄ilo-declaration]: https://www.ilo.org/declaration/lang--en/index.htm
[📄ilo-declaration-img]: https://img.shields.io/badge/ILO_Fundamental_Principles-✓-259D6C.svg?style=flat

[dotenv]: https://github.com/bkeepers/dotenv
