# Installation — Docker

!!! note "Stub"
    Skeleton — headings below are the intended structure, to be filled in.

Run the full toolchain in a container, so the only host requirement is Docker.

## Prerequisites

<!-- TODO: Docker Engine / Desktop install per host OS -->

## 1. Get the image

<!-- TODO: docker pull <image>  OR  build from the provided Dockerfile -->

## 2. Run a build

<!-- TODO: docker run with the project mounted as a volume, working dir,
     example: building an example project -->

## 3. Programming the board

<!-- TODO: USB passthrough caveats (--device / privileged), notes for
     Linux host vs Docker Desktop (no native USB passthrough) -->

## Verify

```bash
docker run --rm <image> anvil doctor
```

<!-- TODO: expected output -->
