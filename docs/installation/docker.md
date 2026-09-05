# Installation — Docker

!!! warning "Does not exist yet"
    There is **no Docker image for the toolchain**, and no Dockerfile to build one
    from. Nothing on this page is usable yet — it is a placeholder so existing
    links do not break.

    Install with [Ubuntu (native)](ubuntu.md) or [WSL2](wsl.md) instead. Both are
    complete and tested.

## Why there is no image

Containers were considered and set aside for now:

- **On Windows**, the toolchain runs under WSL2. Docker Desktop would add a second
  Linux VM on top of that, plus slow bind-mounted project files — real overhead
  for no gain, since WSL2 already provides the Linux environment.
- **On native Linux**, containers cost almost nothing at runtime, but building and
  maintaining an image is work that has not been done, and USB passthrough for
  programming a board needs extra care (`--device`, or privileged mode).

The longer-term direction for reproducibility is **Nix** rather than Docker: it
runs directly on the host with no VM, while still pinning exact versions. See the
[decision log](https://github.com/LogiSmith/toolchain-setup/blob/main/DECISIONS.md)
in the installer repository.

## If you want to build one anyway

Nothing here is official or tested, but the shape is straightforward: start from
`ubuntu:22.04`, run the [installer](https://github.com/LogiSmith/toolchain-setup)
inside it as a non-root user with `--no-board --no-test`, and mount your project
at build time. Programming a board from the container needs the FTDI device passed
in; on Docker Desktop (Windows/macOS) there is no native USB passthrough at all, so
use WSL2 for that.

Contributions are welcome — open an issue on
[Docs](https://github.com/LogiSmith/Docs) or the
[installer repo](https://github.com/LogiSmith/toolchain-setup) first, so the
approach can be agreed before the work.
