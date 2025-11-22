# GBLN Project Documentation Index

## Core Documentation

### Overview
- [00-introduction.md](00-introduction.md) - Project overview and vision
- **01-index.md** - This file
- [02-specification.md](02-specification.md) - Complete GBLN specification v1.1
- [03-architecture.md](03-architecture.md) - Overall project architecture

### Implementation Guides
- [04-rust-implementation.md](04-rust-implementation.md) - Rust reference implementation
- [05-c-implementation.md](05-c-implementation.md) - C FFI foundation
- [06-parser-architecture.md](06-parser-architecture.md) - Parser design patterns

## Language-Specific Guides

### Tier 1: Must-Have (Weeks 3-4)
- [10-python-binding.md](10-python-binding.md) - Python implementation via ctypes
- [11-javascript-binding.md](11-javascript-binding.md) - JavaScript/Node.js via WASM
- [12-swift-binding.md](12-swift-binding.md) - Swift implementation for iOS/macOS
- [13-kotlin-binding.md](13-kotlin-binding.md) - Kotlin implementation for Android/JVM
- [14-go-binding.md](14-go-binding.md) - Go implementation via CGO

### Tier 2: Should-Have (Weeks 5-6)
- [15-java-binding.md](15-java-binding.md) - Java implementation via JNA
- [16-csharp-binding.md](16-csharp-binding.md) - C# implementation via P/Invoke
- [17-cpp-binding.md](17-cpp-binding.md) - C++ native implementation

### Tier 3: Nice-to-Have (Weeks 7-8)
- [18-ruby-binding.md](18-ruby-binding.md) - Ruby implementation via FFI
- [19-php-binding.md](19-php-binding.md) - PHP extension
- [20-perl-binding.md](20-perl-binding.md) - Perl XS extension
- [21-tcl-binding.md](21-tcl-binding.md) - Tcl extension

## Tooling & Ecosystem

### CLI Tools
- [30-formatter.md](30-formatter.md) - GBLN formatter (`gbln fmt`)
- [31-linter.md](31-linter.md) - GBLN linter (`gbln lint`)
- [32-validator.md](32-validator.md) - GBLN validator (`gbln validate`)
- [33-converter.md](33-converter.md) - JSON ↔ GBLN converter

### Editor Support
- [40-vscode-extension.md](40-vscode-extension.md) - VS Code syntax highlighting
- [41-vim-plugin.md](41-vim-plugin.md) - Vim plugin
- [42-zed-extension.md](42-zed-extension.md) - Zed editor extension
- [43-jetbrains-plugin.md](43-jetbrains-plugin.md) - JetBrains IDEs plugin (IntelliJ, PyCharm, etc.)
- [44-sublime-package.md](44-sublime-package.md) - Sublime Text package

## Testing & Quality

### Testing Strategy
- [50-testing-strategy.md](50-testing-strategy.md) - Overall testing approach
- [51-unit-tests.md](51-unit-tests.md) - Unit test guidelines
- [52-integration-tests.md](52-integration-tests.md) - Integration test suites
- [53-cross-language-tests.md](53-cross-language-tests.md) - Cross-language compatibility

### Benchmarking
- [60-benchmark-suite.md](60-benchmark-suite.md) - Benchmark methodology
- [61-memory-benchmarks.md](61-memory-benchmarks.md) - Memory usage comparison
- [62-parse-speed-benchmarks.md](62-parse-speed-benchmarks.md) - Parse speed comparison
- [63-comparison-formats.md](63-comparison-formats.md) - vs JSON, YAML, TOML, TOON

## Examples & Use Cases

### Basic Examples
- [70-basic-examples.md](70-basic-examples.md) - Simple GBLN examples
- [71-config-files.md](71-config-files.md) - Configuration file examples
- [72-api-responses.md](72-api-responses.md) - API response examples
- [73-database-export.md](73-database-export.md) - Database export examples

### Advanced Examples
- [74-nested-structures.md](74-nested-structures.md) - Complex nested data
- [75-iot-data.md](75-iot-data.md) - IoT sensor data examples
- [76-mobile-apps.md](76-mobile-apps.md) - Mobile app integration

### Real-World Use Cases
- [80-use-case-config.md](80-use-case-config.md) - Configuration management
- [81-use-case-api.md](81-use-case-api.md) - API data exchange
- [82-use-case-iot.md](82-use-case-iot.md) - IoT device communication
- [83-use-case-mobile.md](83-use-case-mobile.md) - Mobile app data storage
- [84-use-case-embedded.md](84-use-case-embedded.md) - Embedded systems

## Community & Contribution

### Contributing
- [90-contributing.md](90-contributing.md) - How to contribute
- [91-code-style.md](91-code-style.md) - Code style guidelines
- [92-pull-request-process.md](92-pull-request-process.md) - PR guidelines
- [93-issue-templates.md](93-issue-templates.md) - Issue reporting templates

### Community
- [94-community.md](94-community.md) - Community resources
- [95-faq.md](95-faq.md) - Frequently asked questions
- [96-roadmap.md](96-roadmap.md) - Project roadmap
- [97-changelog.md](97-changelog.md) - Version changelog

## Release & Deployment

### Package Management
- [100-cargo-package.md](100-cargo-package.md) - Rust crate publishing
- [101-npm-package.md](101-npm-package.md) - NPM package publishing
- [102-pip-package.md](102-pip-package.md) - PyPI package publishing
- [103-maven-package.md](103-maven-package.md) - Maven Central publishing
- [104-nuget-package.md](104-nuget-package.md) - NuGet package publishing
- [105-cocoapods-package.md](105-cocoapods-package.md) - CocoaPods spec
- [106-gradle-package.md](106-gradle-package.md) - Gradle/Maven publishing

### CI/CD
- [110-ci-setup.md](110-ci-setup.md) - Continuous integration setup
- [111-github-actions.md](111-github-actions.md) - GitHub Actions workflows
- [112-release-process.md](112-release-process.md) - Release process
- [113-versioning.md](113-versioning.md) - Version numbering strategy

## Marketing & Launch

### Website & Docs
- [120-website.md](120-website.md) - gbln.dev website structure
- [121-playground.md](121-playground.md) - Interactive playground
- [122-api-docs.md](122-api-docs.md) - API documentation generation

### Launch Strategy
- [130-launch-plan.md](130-launch-plan.md) - Launch strategy
- [131-marketing-materials.md](131-marketing-materials.md) - Marketing assets
- [132-social-media.md](132-social-media.md) - Social media strategy
- [133-community-building.md](133-community-building.md) - Community growth

## Reference

### Specifications
- [200-grammar-ebnf.md](200-grammar-ebnf.md) - EBNF grammar reference
- [201-type-system.md](201-type-system.md) - Type system details
- [202-validation-rules.md](202-validation-rules.md) - Validation rules reference
- [203-error-messages.md](203-error-messages.md) - Error message catalog

### Best Practices
- [210-best-practices.md](210-best-practices.md) - GBLN best practices
- [211-security.md](211-security.md) - Security considerations
- [212-performance.md](212-performance.md) - Performance optimization
- [213-migration.md](213-migration.md) - Migration from JSON/YAML

## Quick Links

### For Developers
- New to GBLN? Start with [00-introduction.md](00-introduction.md)
- Want to implement? Read [04-rust-implementation.md](04-rust-implementation.md) and [06-parser-architecture.md](06-parser-architecture.md)
- Need language-specific guide? See sections 10-21
- Want to contribute? Check [90-contributing.md](90-contributing.md)

### For Users
- Basic usage? See [70-basic-examples.md](70-basic-examples.md)
- Specific use case? Browse sections 80-84
- Have questions? Read [95-faq.md](95-faq.md)
- Report issues? Use [93-issue-templates.md](93-issue-templates.md)

### For Project Leads
- Track progress: [96-roadmap.md](96-roadmap.md)
- Plan release: [112-release-process.md](112-release-process.md)
- Launch prep: [130-launch-plan.md](130-launch-plan.md)

---

## Document Status

| Doc ID | Status | Priority | Assignee |
|--------|--------|----------|----------|
| 00-03 | ✅ Complete | Critical | Core Team |
| 04-06 | 🚧 In Progress | Critical | Rust Team |
| 10-14 | 📋 Planned | High | Binding Teams |
| 15-21 | 📋 Planned | Medium | Binding Teams |
| 30-42 | 📋 Planned | Medium | Tools Team |
| 50-63 | 📋 Planned | High | QA Team |
| 70-84 | 📋 Planned | Medium | Docs Team |
| 90-97 | 📋 Planned | Low | Community Team |
| 100-113 | 📋 Planned | High | DevOps Team |
| 120-133 | 📋 Planned | Medium | Marketing Team |
| 200-213 | 📋 Planned | Low | Docs Team |

**Legend:**
- ✅ Complete
- 🚧 In Progress
- 📋 Planned
- ⏸️ On Hold
- ❌ Cancelled

---

*Last Updated: 2025-01-20*
*Version: 1.0.0*
