---
tags:
  - Docker
---

# Methods

## [Emulation](https://docs.docker.com/build/guide/multi-platform/#build-using-emulation)

Using **QEMU** which is provided with **Docker Desktop**.

The image used in `FROM` must be available on the target platform.

## [Cross-compilation](https://docs.docker.com/build/guide/multi-platform/#build-using-cross-compilation)

Using image provided capabilities to cross-compile.

`GOOS` and `GOARCH` for Go `build` command #Go.

Use [`ARG`s](https://docs.docker.com/engine/reference/builder/#automatic-platform-args-in-the-global-scope) provided by Docker:

Set using `--platform` flag in `build` command.

- `BUILDPLATFORM` - platform of the host running the build
- `BUILDOS` - OS part of the `BUILDPLATFORM`
- `BUILDARCH` - arch part of the `BUILDPLATFORM`
- `TARGETPLATFORM` - target platform
- `TARGETOS` - OS part of the `TARGETPLATFORM`
- `TARGETARCH` - architecture part of the `TARGETPLATFORM`

# Examples

`go` uses different `Dockerfile` for each arch/OS variant. See: [docker library/golang](https://github.com/docker-library/golang)

`hello-world` uses the same approach: [docker-library/hello-world](https://github.com/docker-library/hello-world)

# References

- [Multi-platform images | Docker Docs](https://docs.docker.com/build/building/multi-platform/)
- [Build with Docker | Docker Docs](https://docs.docker.com/build/guide/)
