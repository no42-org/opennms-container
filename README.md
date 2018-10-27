# OpenNMS Container Image Build

This repository contains the container build instructions for the following OpenNMS components (projects):

* OpenNMS Horizon
* OpenNMS Minion

The build is divided in three steps:

1. `.circleci/build.sh`: Build the container image
2. `.circleci/tag.sh`: Tag container images with registry and version tags
3. `.circleci/push.sh`: Push container images to a registry

These tasks are executed in the project context which is `projects/horizon` for OpenNMS Horizon and `projects/minion` for the Minion application.

Each project can container build specific configurations which are persisted in `projects/<project-name>/config.sh`.

A local build can be done locally by just running the three scripts above in the project context.

Example to build, tag and push the Horizon Image

```bash
git clone https://github.com/no42-org/opennms-container.git
cd opennms-container

cd projects/horizon
./build_container_image.sh
../../.circleci/tag.sh
../../.circleci/push.sh
```

The credentials to push to a remote container registry are provided

a) Environment variable `CONTAINER_REGISTRY_LOGIN` and `CONTAINER_REGISTRY_PASS`
b) If not set the local registry configuration is used and tested with `docker login`

By default the `docker.io` and `opennms` as a repository is set.

* CircleCI Status: [![CircleCI](https://circleci.com/gh/no42-org/opennms-container.svg?style=svg)](https://circleci.com/gh/no42-org/opennms-container)
* [DockerHub](https://hub.docker.com/r/no42org)

## Configure Docker Build

Running within a container infrastructure requires customizations to some extend, e.g.

* Running with Oracle or OpenJDK
* Custom OpenNMS RPM packages, e.g. `opennms-plugin-protocol-cifs`, `opennms-webapp-hawtio`
* Pushing to a private registry
* Installing from private mirror servers
* Tagging of container images

These parameters can be modified in the `config.sh` in each individual project.

## CircleCi Pipeline

Building always all container images on *any* commit in the repository cost time and resources.
This repository builds only container images when the commit has changes inside of the project directory.

```bash
if git diff --name-only HEAD^...HEAD | grep "^projects/${PROJECT}"; then
  echo "Changes here, run the build"
  # ... trigger build, tag and push in project
else
  echo "No changes detected"
  exit 0
fi
```

A successful container build produces an artifact which is persited in `projects/<project-name>/images/image.oci`.
CircleCI will stores this artifact and can be downloaded.
Once the image is downloaded it can be loaded with

```bash
docker load -i <path/to/image.oci>
```
