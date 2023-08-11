[![Build CI](https://github.com/scottharwell/cloud-ee/actions/workflows/main.yml/badge.svg)](https://github.com/scottharwell/cloud-ee/actions/workflows/build.yml)

# Ansible Cloud Content Execution Environment

This is an Ansible execution environment that contains collections to automate cloud infrastructure and supporting collections to help incubate cloud content.  It is automatically published to [quay.io](https://quay.io/repository/scottharwell/cloud-ee) when new versions of this definition are created.

## Dependencies

The following are Dependencies for the build host/system if you intend to build this EE locally.  These **are not** requirements for the collections or dependencies published inside of the container/ee.

* ansible-builder>=3.0.0

## Using the Execution Environment

### Ansible Automation Platform

Follow these steps to use the execution environment in Ansible Automation Platform:

1. Login to Ansible Automation Controller.
2. Click Execution Environments in the left menu.
3. Click the "Add" button.
4. Fill in the name field: `Cloud EE`
5. Fill in the image field: `quay.io/scottharwell/cloud-ee:latest`
   * You can change `latest` to a version tag to ensure that changes to the EE don't affect your automation without your explicit action.
6. Optional: Change the pull field `Always` so that the latest version of the container is pulled if you are using the `latest` tag.

### Ansible CLI

You can reference the execution environment when running your automation locally with `ansible-navigator`.  Use the documentation for `ansible-navigator` to determine which flags to set for your playbook.  However, the three important flags for using an execution environment are:

* `--ee`: Tells Ansible Navigator to use an execution environment
* `--ce`: Tells Ansible Navigator which container engine to use
* `--eei`: Tells Ansible Navigator which image to use

The `--pp` flag is also useful to tell Ansible Navigator to attempt to pull the container on each run.  This is useful since this execution environment is updated frequently.

```bash
ansible-navigator run create_network.yml \
--pae false \
--mode stdout \
--ee true \
--ce docker \
--pp always \
--eei quay.io/scottharwell/cloud-ee:latest
```

### Using Container Engines

You may pull the container without building it locally by running the pull command.  Use `podman` or `docker` based on the container engine that you use.

```bash
podman pull quay.io/scottharwell/cloud-ee
```

```bash
docker pull quay.io/scottharwell/cloud-ee
```

Then, you may enter the container and use it directly.

```bash
docker run -it \
--rm \
--pull always \
quay.io/scottharwell/cloud-ee:latest \
/bin/bash
```

Once inside the container, you can use the Ansible CLI tools.  Below is an example of checking the collections installed in the container.

```bash
ansible-galaxy collection list
```

## Included Content

### Ansible Collections

The following collections are included in this execution environment.

| Collection           | Description                                                 |
| -------------------- | ----------------------------------------------------------- |
| `amazon.aws`         | Used for AWS automation.                                    |
| `amazon.cloud`       | Newer AWS collection using the cloud control API.           |
| `ansible.utils`      | Ansible general utilities.                                  |
| `ansible.controller` | Used for Ansible Controller automation.                     |
| `ansible.windows`    | Windows collection for automating Windows servers on Azure. |
| `awx.awx`            | Used for AWX automation.                                    |
| `azure.azcollection` | Used for Azure automation.                                  |
| `cloud.terraform`    | Used for automating Terraform with Ansible.                 |
| `community.aws`      | Used for AWS automation.                                    |
| `community.general`  | Used for Proxmox and other community automation.            |
| `google.cloud`       | Used for Google Cloud automation.                           |
| `linode.cloud`       | Used for Linode automation.                                 |
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

| CLI Command | venv Name    | Activation Command                            |
| ----------- | ------------ | --------------------------------------------- |
| `oci`       | `oracle-cli` | `source /home/runner/oracle-cli/bin/activate` |

## Building the Execution Environment

This execution environment is built from downstream containers.  You will need to have a [Red Hat account](https://developers.redhat.com/blog/2016/03/31/no-cost-rhel-developer-subscription-now-available) (a free developer account will work) and be logged in to the Red Hat registry in order for this build process to work. Your Red Hat account username and password will be requested when you run this command:

```bash
docker login registry.redhat.io
```

The execution environment also pulls collections from Automation Hub and requires that an [Automation Hub token](https://console.redhat.com/ansible/automation-hub/token) is supplied via the `ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_TOKEN` environment variable.  Ensure that the environment variable does not have any whitespace or extra characters when you set the variable.

### Intel or AMD `amd64` Architectures

Ansible Builder will function out-of-the-box on `amd64` platforms with Podman or Docker.

1. Run `git clone https://github.com/scottharwell/cloud-ee.git` to clone this repository.
2. Run `cd cloud-ee` to change into the repository directory.
3. Ensure that you have logged in to the Red Hat container registry.
4. Ensure that you have set the `ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_TOKEN` environment variable.
5. Run `ansible-builder build --build-arg ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_TOKEN=$ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_TOKEN -t <YOUR_TAG_HERE>`.

### Apple Silicon or `arm64` Architectures

The `azure.azure_rm` collection and Azure CLI use Python dependencies that do not have `arm64` equivalents at this time; a pure `arm64` build will fail during the build phase.  Therefore, building a *native* `arm64` container is blocked.

However, it is possible to build an `arm64` compatible container that emulates `amd64` to circumvent this issue and eliminates the need to pass a platform flag to the container runtime.  This requires Docker to build EEs. Podman does not function with these steps on an Apple Silicon Mac at this time.  This process will build a multi-platform container on ARM architectures that will then function on either an `amd64` or `arm64` platform.

1. Run `git clone https://github.com/scottharwell/cloud-ee.git` to clone this repository.
2. Run `cd cloud-ee` to change into the repository directory.
3. Run `ansible-builder create --output-filename Dockerfile` to create the `context` directory.
4. Run `gsed -i 's/FROM/FROM --platform=linux\/amd64/' context/Dockerfile` to ensure that the container will build properly.
5. Run `cd context`.
6. Ensure that you have logged in to the Red Hat container registry.
7. Ensure that you have set the `ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_TOKEN` environment variable.
8. Run the commands below to set your own container registry, version, and then build the container.

   ```bash
   export REGISTRY=quay.io/scottharwell
   export VERSION=1.0.0
   docker buildx build --no-cache --platform linux/arm64,linux/amd64 --build-arg ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_TOKEN=$ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_TOKEN -t $REGISTRY/cloud-ee:$VERSION -t $REGISTRY/cloud-ee:latest --push .
   ```

9. Run `docker pull $REGISTRY/cloud-ee:$VERSION` to pull the image to your machine.

You may now run Ansible playbooks with `ansible-navigator` natively on your Apple Silicon Mac.
