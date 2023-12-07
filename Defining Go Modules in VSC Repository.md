---
tags:
  - Go
---

# Root Directory

If a module is defined in the repository root directory or in major version subdirectory, then each version tag name is equal to the corresponding version

For example, the module `golang.org/x/text` is defined in the root directory of its repository, so the version `v0.3.2` has the tag `v0.3.2` in that repository

# Subdirectory

If a module is defined in a subdirectory within the repository, that is, the [module subdirectory](https://go.dev/ref/mod#glos-module-subdirectory) portion of the module path is not empty, then each tag name must be prefixed with the module subdirectory, followed by a slash

For example, the module `golang.org/x/tools/gopls` is defined in the `gopls` subdirectory of the repository with root path `golang.org/x/tools`

The version `v0.4.0` of that module must have the tag named `gopls/v0.4.0` in that repository

# References

- [Go Modules Reference - The Go Programming Language](https://go.dev/ref/mod#vcs)
