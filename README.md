[![Build CI](https://github.com/scottharwell/cloud-ee/actions/workflows/build.yml/badge.svg)](https://github.com/scottharwell/cloud-ee/actions/workflows/build.yml)

# Cloud Content Execution Environment

This is a simple execution environment definition for cloud deployment testing with hyperscaler content collections and other collections and roles from the Ansible team to help incubate cloud content. It includes the following.

## Included Content

### Ansible Collections

The following collections are included in this execution environment.

| Collection           | Description                                                 |
| -------------------- | ----------------------------------------------------------- |
| `amazon.aws`         | Used for AWS automation.                                    |
| `amazon.cloud`       | Newer AWS collection using the cloud control API.           |
| `ansible.windows`    | Windows collection for automating Windows servers on Azure. |
| `awx.awx`            | Used for AWX/Automation Controller automation.              |
| `azure.azcollection` | Used for Azure automation.                                  |
| `cloud.terraform`    | Used for automating Terraform with Ansible.                 |
| `community.aws`      | Used for AWS automation.                                    |
| `community.general`  | Used for Proxmox and other community automation.            |
| `google.cloud`       | Used for Google Cloud automation.                           |
| `oracle.oci`         | Used for OCI automation.                                    |

### CLI Tools

The following CLI tools are included in this execution environment.

| Cloud                       | CLI Command |
| --------------------------- | ----------- |
| Amazon Web Services         | `aws`       |
| Microsoft Azure             | `az`        |
| Google Cloud Platform       | `gcloud`    |
| Oracle Cloud Infrastructure | `oci`       |
| Terraform                   | `terraform` |

The following CLI tools are installed into Python virtual environments (venv).  To access them within the venv, run the activation command below.

| CLI Command | venv Name | Activation Command            |
| ----------- | --------- | ----------------------------- |
| `oci`       | `oci-cli` | `source oracle-cli/bin/activate` |

## Building the Execution Environment

### Intel or AMD `amd64` on Linux or Mac

Ansible Builder will function out-of-the-box on `amd64` platforms with Podman or Docker.

1. Clone this repo.
2. Update the included files in this repository with the variables to include in the EE.
   * `requirements.yml` for Ansible collections and roles.
   * `requirements.txt` for Python dependencies.
3. Run `ansible-builder build -t <YOUR_TAG_HERE>`.

### Apple Silicon

This requires Docker to build EEs. Podman does not function with these steps on an Apple Silicon Mac at this time.  This process will build a multi-platform container on an Apple Silicon Mac that will then function on either an `amd64` or `arm64` platform.

1. Clone this repository.
2. Run `ansible-builder create` to create the `context` directory.
3. Change into the newly created `context` directory.
4. Run `docker buildx`, which is capable of building multi-architecture containers, to build an image that can be used on both `amd64` and `arm64/aarch64` (Apple Silicon and other ARM) CPUs.

In the example below, change the `REGISTRY` and `VERSION` variables to match your container registry that you intend to push to and a version number to tag the build.

```bash
export REGISTRY=quay.io/scottharwell
export VERSION=1.0.0
docker buildx build --platform linux/arm64,linux/amd64 -t $REGISTRY/cloud-ee:$VERSION -t $REGISTRY/cloud-ee:latest --push .
```

Once the container is deployed to your container registry, then you can pull the EE down to your Apple Silicon Mac to run natively, or to Ansible Controller running on `amd64` machines.
