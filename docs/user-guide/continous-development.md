---
description: How to use Continuous Development in Elastisys Compliant Kubernetes, the security-focused Kubernetes distribution.
---

# Continuous Development

When developing on Kubernetes, it can be time-consuming to manually run commands to build new images,
push images to a container registry, update Kubernetes manifests, and deploy new manifests to the cluster with
each source-code change.

[Skaffold](https://github.com/GoogleContainerTools/skaffold) is a tool that can be used to ease this process.
Skaffold will automate the process of building, pushing and deploying new images based on changes to the source-code.

Skaffold requires minimal setup as the tool has no cluster-side components, and will automatically detect the configuration to use.

## Installing Skaffold

The Skaffold CLI can be installed by downloading and installing the latest release. Instructions can be found in the [Skaffold
documentation](https://skaffold.dev/docs/install/):

```sh
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/v2.0.1/skaffold-linux-amd64 && \
sudo install skaffold /usr/local/bin/
```

## Getting started with Skaffold

!!!note
    Skaffold will use the active `KUBECONFIG` to authenticate to the Kubernetes cluster.

If you havent't done so already, clone the user demo:

```bash
git clone https://github.com/elastisys/compliantkubernetes/
cd compliantkubernetes/user-demo
```

<!-- TODO: I didn't read the docs thingie? -->

### Initialize Skaffold

```bash
skaffold init
```

This command will scan the current project for images to build and Kubernetes manifests to deploy. For each image that
Skaffold finds you will be prompted to specify how to build them, or if they are not built by the current project.

The first image skaffold finds is busybox, however this image is not built from this project, so choose
`None (image not built from these sources)`

![Skaffold init Output1](/compliantkubernetes/user-guide/img/skaffold-busybox.png)

The second image Skaffold finds is the user-demo image and this image is built from the Dockerfile.

![Skaffold init Output2](/compliantkubernetes/user-guide/img/skaffold-user-demo.png)

Skaffold then asks for which resources we want to create Kubernetes resources, but as the image already has a helm-chart
this can be skipped by pressing enter.

Skaffold will then create the `skaffold.yaml` file containing our configuration. Skaffold will also automatically detect the
helm-chart that deploys the `user-demo` image.

### Developing

To start developing using Skaffold run:

```bash
skaffold dev
```

When you run `skaffold dev`, Skaffold will first build, and deploy all of the artifacts specified in `skaffold.yaml`.
Skaffold will then begin monitoring all source file dependencies for all artifacts specified in the project and rebuild
the associated artifacts and redeploy the new changes to your cluster as changes are made to these source files.
So any changes made to the source-files will automatically be updated in the cluster.

When starting `skaffold dev`, the logs of the deployed artifacts will automatically be directed to the console, which makes it
easy to debug the application in the cluster.

### Advanced

- Skaffold also supports port-forwarding to the deployed resources. This can be enabled by adding the `--port-forward` flag to the
  dev command:

    ```bash
    skaffold dev --port-forward
    ```

    Skaffold will then automatically port-forward local ports to the containers running in the cluster.

- Skaffold supports [File Sync](https://skaffold.dev/docs/pipeline-stages/filesync/) which will avoid rebuilding of containers by
  automatically updating images inside the running container instead of re-building it. This is setup inside the `skaffold.yaml` file.

### Clean-up

To stop Skaffold `ctrl + c` can be used and will trigger Skaffold to stop listening for changes and
clean-up the deployed artifacts from the cluster.

This can also be triggered by running:

```bash
skaffold delete
```
