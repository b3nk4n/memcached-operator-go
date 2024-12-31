# memcached-operator

A simple K8s operator for Memcached using Operator SDK in Go.

The project is meant for learning purposes only, using the course [IBM CO0201EN - Kubernetes Operators Intermediate](https://apps.cognitiveclass.ai/learning/course/course-v1:CognitiveClass+CO0201EN+v2/home). Thus, it is not meant for production use.


## Getting Started

### Prerequisites
- go version v1.22.0+
- docker version 17.03+.
- kubectl version v1.11.3+.
- Access to a Kubernetes v1.11.3+ cluster.

### Development

You can run the operator as a program executing outside the Kubernetes cluster.
This might be done for development purposes or for security concerns of the data contained in the cluster.
The Makefile contains the target `make install run` to run the operator locally, if needed.

### To Deploy on the cluster
**Build and push your image to the location specified by `IMG`:**

```sh
make docker-build docker-push IMG=<some-registry>/memcached-operator:tag
```

**NOTE:** This image ought to be published in the personal registry you specified.
And it is required to have access to pull the image from the working environment.
Make sure you have the proper permission to the registry if the above commands don’t work.

**Install the CRDs into the cluster:**

```sh
make install
```

**Deploy the Manager to the cluster with the image specified by `IMG`:**

```sh
make deploy IMG=<some-registry>/memcached-operator:tag
```

> **NOTE**: If you encounter RBAC errors, you may need to grant yourself cluster-admin
privileges or be logged in as admin.

You can then verify the created resources, such as using:

```sh
kubectl get crds
kubectl --namespace memcached-operator-system get deployments
kubectl --namespace memcached-operator-system get pods
```

**Create instances of your solution**
You can apply the samples (examples) from the config/sample:

```sh
kubectl apply -k config/samples/
```

>**NOTE**: Ensure that the samples has default values to test it out.

### To Uninstall
**Delete the instances (CRs) from the cluster:**

```sh
kubectl delete -k config/samples/
```

**Delete the APIs(CRDs) from the cluster:**

```sh
make uninstall
```

**Un-deploy the controller from the cluster:**

```sh
make undeploy
```

## Project Distribution

Following are the steps to build the installer and distribute this project to users.

1. Build the installer for the image built and published in the registry:

```sh
make build-installer IMG=<some-registry>/memcached-operator:tag
```

NOTE: The makefile target mentioned above generates an 'install.yaml'
file in the dist directory. This file contains all the resources built
with Kustomize, which are necessary to install this project without
its dependencies.

2. Using the installer

Users can just run kubectl apply -f <URL for YAML BUNDLE> to install the project, i.e.:

```sh
kubectl apply -f https://raw.githubusercontent.com/<org>/memcached-operator/<tag or branch>/dist/install.yaml
```

## Create and install a bundle using OLM

Create a bundle:

```sh
export DOCKERUSER=b3nk4n
export VERSION=0.2.0
export IMG=docker.io/$DOCKERUSER/memcached-operator:v$VERSION
export BUNDLE_IMG=docker.io/$DOCKERUSER/memcached-operator-bundle:v$VERSION

make bundle
make bundle-build bundle-push
```

And verify that the bundle is correct:

```sh
operator-sdk bundle validate docker.io/$DOCKERUSER/memcached-operator-bundle:v$VERSION
```

And then use the following OLM commands to install the bundle (which can take a while):
```sh
operator-sdk olm install
operator-sdk olm status
operator-sdk run bundle docker.io/$DOCKERUSER/memcached-operator-bundle:v$VERSION --timeout 30m
operator-sdk olm status
```

## Key takeaways for writing Kubernetes Operators

1. Information flows **in** from the requested object's `spec` and **out** to its `status`.
2. You should iterate on reconciliation by doing a **single step** and then requeueing.


## License

Copyright 2024.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

