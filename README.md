# kube-prometheus-builder

Docker image to help build kube-prometheus.

Instead of having to install go and the jsonnet stuff locally you can use this image when customizing your kube-prometheus installation.

See <https://github.com/prometheus-operator/kube-prometheus/blob/main/docs/customizing.md> for details.

## Usage

1. Obtain [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus) and make required changes.
1. Acquire `build.sh` from kube-prometheus: `wget https://raw.githubusercontent.com/prometheus-operator/kube-prometheus/main/build.sh -O build.sh`
1. Create `create_manifests.sh` with this content:

```bash
# Update the image
docker pull ghcr.io/roelandvanbatenburg/kube-prometheus-builder
# Update jsonnet-bundler inside the docker container, this creates a vendor directory
docker run --rm -v "$(pwd):$(pwd)" --workdir "$(pwd)" ghcr.io/roelandvanbatenburg/kube-prometheus-builder jb update
# Run the build script, this creates a manifests directory
docker run --rm -v "$(pwd):$(pwd)" --workdir "$(pwd)" ghcr.io/roelandvanbatenburg/kube-prometheus-builder ./build.sh config.jsonnet
```

Run it using:

```bash
chmod +x create_manifests.sh
./create_manifests.sh
```

Finally, follow the [kube-prometheus instructions](https://github.com/prometheus-operator/kube-prometheus/blob/main/docs/customizing.md#apply-the-kube-prometheus-stack) on applying the stack.

```bash
kubectl apply --server-side -f manifests/setup
kubectl wait \
 --for condition=Established \
 --all CustomResourceDefinition \
 --namespace=monitoring
kubectl apply -f manifests/
```
