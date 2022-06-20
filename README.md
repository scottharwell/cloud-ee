[![Build CI](https://github.com/scottharwell/cloud-ee/actions/workflows/build.yml/badge.svg?branch=v1.0.0)](https://github.com/scottharwell/cloud-ee/actions/workflows/build.yml)

# Cloud Content Execution Environment

This is a simple execution environment definition for cloud deployment testing with hyperscaler content collections and other collections and roles from the Ansible team to help incubate cloud content.

## Building on Intel or AMD Linux or Mac

Ansible Builder will function out-of-the-box on `amd64` platforms with Podman or Docker.

1. Clone this repo.
2. Update the included files in this repository with the variables to include in the EE.
   * `requirements.yml` for Ansible collections and roles.
   * `requirements.txt` for Python dependencies.
3. Run `ansible-builder build -t <YOUR_TAG_HERE>`.

## Building on an M1 Mac

This requires Docker to build EEs. Podman does not function with these steps on an M1 Mac at this time.

1. Clone this repository.
2. Run `ansible-builder create` to create the `context` directory.
3. Change into the newly created `context` directory.
4. Open the newly created file `Containerfile`.
5. Add `--platform=linux/amd64` to each `FROM` statement since most quay images are only built for the `linux/amd64` platform.
    * Statements will look like: `FROM --platform=amd64 $EE_BASE_IMAGE as galaxy`
6. Using `docker buildx`, which is capable of building multi-architecture containers, to build an image that can be used on both `amd64` and `ARM64/aarch64` (M1 and other ARM) CPUs.

```bash
export REGISTRY=quay.io/scottharwell
export VERSION=1.0.0
docker buildx build --platform linux/arm64,linux/amd64 -t $REGISTRY/cloud-ee:$VERSION -t $REGISTRY/cloud-ee:latest --push .
```

Once the container is deployed to your container registry, then you can pull the EE down to your M1 Mac natively, or to Ansible Controller running on x86 machines.