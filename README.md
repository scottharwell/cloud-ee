[![Build CI](https://github.com/scottharwell/cloud-ee/actions/workflows/build.yml/badge.svg)](https://github.com/scottharwell/cloud-ee/actions/workflows/build.yml)

# Cloud Content Execution Environment

This is execution environment definition for automation against cloud infrastructure providers and other collections and roles from the Ansible team to help incubate cloud content.

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
| `linode.cloud`       | Used for Linode Cloud automation.                           |
| `oracle.oci`         | Used for OCI automation.                                    |
| `vultr.cloud`        | Used for Vultr automation.                                  |

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

| CLI Command | venv Name    | Activation Command               |
| ----------- | ------------ | -------------------------------- |
| `oci`       | `oracle-cli` | `source oracle-cli/bin/activate` |

## Building the Execution Environment

This execution environment is built from downstream containers.  You will need to have a [Red Hat account](https://developers.redhat.com/blog/2016/03/31/no-cost-rhel-developer-subscription-now-available) (a free developer account will work) and be logged in to the Red Hat registry in order for this build process to work.

```bash
docker login registry.redhat.io
```

### Intel or AMD `amd64` on Linux or Mac

Ansible Builder will function out-of-the-box on `amd64` platforms with Podman or Docker.

1. Clone this repo.
2. Update the included files in this repository with the variables to include in the EE.
   * `requirements.yml` for Ansible collections and roles.
   * `requirements.txt` for Python dependencies.
3. Run `ansible-builder build -t <YOUR_TAG_HERE>`.

### Apple Silicon

This requires Docker to build EEs. Podman does not function with these steps on an Apple Silicon Mac at this time.  This process will build a multi-platform container on an Apple Silicon Mac that will then function on either an `amd64` or `arm64` platform.

1. Run `git clone https://github.com/scottharwell/cloud-ee.git` to clone this repository.
2. Run `cd cloud-ee` to change into the repository directory.
3. Run `ansible-builder create --output-filename Dockerfile` to create the `context` directory.
4. Run `gsed -i 's/FROM/FROM --platform=linux\/amd64/' context/Dockerfile` to ensure that the container will build properly.
5. Run `cd context`.
6. Run the following commands below to set your own container registry, version, and then build the container.

   ```bash
   export REGISTRY=quay.io/scottharwell
   export VERSION=1.0.0
   docker buildx build --no-cache --platform linux/arm64,linux/amd64 -t $REGISTRY/cloud-ee:$VERSION -t $REGISTRY/cloud-ee:latest --push .
   ```

7. Run `docker pull $REGISTRY/cloud-ee:$VERSION` to pull the image to your machine.

You may now run Ansible playbooks with `ansible-navigator` natively on your Apple Silicon Mac.
