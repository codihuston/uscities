# Getting Started

This file describes how to develop on this project.

## For Developers

This document will tell you how to begin developing on this boilerplate.
Documentation that details exactly either component in this project will be
contained in the `docs` directory of each respective component. This only
serves as an entry point into developing on the application it self; that is,
how to start the application(s).

Currently, you can develop in this project by starting it using the
following tooling:

1. Using `skaffold` (multi-container)

In the future, there may be more ways to develop on this project.

## Developing Using Skaffold

This method relies on Kubernetes to run this project in a multi-container
environment on any platform that supports Kubernetes in some form or fashion. The benefit of this is, we take this straight to production at
scale with a few changes!

### Prerequisites

1. Kubernetes is installed ([Docker Desktop](https://www.docker.com/products/docker-desktop) with Kubernetes Enabled)

    - Docker > Settings > Kubernetes > Check, Enable Kubernetes > Apply & Restart
    - A [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) setup with `kubectl` should suffice as well
    - Ultimately, when `kubectl` is operational
    > NOTE: You may want to go to Docker > Settings > Resources and expand the
    available resources to the K8s cluster. In the next section, you can
    take advantage of those resources when you copy k8s configs to the
    dev directory

2. [Skaffold](https://skaffold.dev/) is installed

### Starting the Project

1. In order to begin, we must set up [ingress-nginx](https://kubernetes.github.io/ingress-nginx/). It is possible that you may already have one enabled on your machine. If that is the case, you may skip this setup.
Otherwise, use the provided `ingress-nginx-example.yml` file as such:

    ```cmd
    # to init / update
    kubectl apply -f ./k8s/ingress-nginx-example.yml

    # to delete (takes time)
    kubectl delete -f ./k8s/ingress-nginx-example.yml
    ```

    This will install the `ingress controller`, which we will later configure
    using an `ingress service`. The service will route traffic to the app(s)
    as per the rules defined in our `k8s-dev\ingres-service.yml` file.

    > Note: Once you restart your computer, you may have to restart this
    controller in order to view your web app via your browser

1. Copy the contents of the `k8s/examples` directory to the `k8s/dev` directory

    > Note: If you want to configure environment variables or other default
    configurations, do so in the destination directory mentioned above.
    The defaults work out-of-box for `Windows w/ Docker Desktop Kubernetes`.
    If you are operating in a different environment, you will need to make
    changes to the `hostPath` key in the `mongodb-deployment.yml`. See that file
    for additional instructions

    1. Change the resource allocation in the deployment files to your liking.
    The out-of-box configuration might be very slow, especially for the
    client and server services

    1. Replace the `REPLACE_ME_WITH_YOUR_DOCKER_ID` token in that file with
    your name (if you want to use your own dockerhub). Not a strict requirement
    if not

1. Copy `skaffold/skaffold-example.yml` to `skaffold/skaffold.yml`; replace the
`REPLACE_ME_WITH_YOUR_DOCKER_ID` in this file as well

1. Install dependencies for each project.

    ```cmd
    cd server
    yarn install

    cd client
    yarn install
    ```

    > Note that the `node_modules` directories are excluded during
    the docker image process via `.dockerignore`, so you shouldn't see any
    performance drops as as result of installing dependencies

1. From the `skaffold` directory in the project root, run the `Skaffold` config
file

    ```cmd
    skaffold dev
    ```

    > Note: You may see an error printed by the API server indicating a database error. This will be addressed in the next step.

    > Note: You can exit the `Skaffold` via SIGINT (ctrl+c). Exiting `Skaffold`
    will destroy any Kubernetes Objects defined in `k8s-dev`. The Persistent
    Volume Claims should persist between `Skaffold` instances.

    You can run the following commands to get the status of the
    services/deployments/pods:

    ```cmd
    kubectl get ingress
    kubectl get services
    kubectl get deployments
    kubectl get pods

    kubectl describe service|deployment|pod <object_name>
    ```

    After the deployments are ready, the Kubernetes cluster is served at `localhost` on port `:8080` or `:443` (`WARNING:` no tls/ssl cert is used in development!). These ports are configured by default using `ingress-nginx-config.yml`. If you need to change those for any reason, make those changes to a new file locally named `ingress-nginx.yml`, and do not commit it to source (that filename is excluded from this repo by default).

    This step will apply the kubernetes config files from `k8s-dev` into the
    kubernetes cluster. This process works like so:

    1. Initialize the Kubernetes Objects as per their definitions
    1. Builds a docker image locally from each of the `Dockerfile.dev` files (if any) for the services being developed on in this project
    1. Skaffold will listen for file changes and attempt to apply them to the containers without having to re-build them (if possible) and automatically rollout the changes to each of the deployments. These live updates might take a little more time than developing without `Skaffold`

    The applications in this project are served on `localhost:8080` as follows:

    1. Client / front-end app: `localhost:8080`
    1. API Server: `localhost:8080/api`
    1. Redis Server: you can access it on `localhost:6379` out-of-box, or
    via the k8s pod itself. You can change this in your k8s deployment file
    1. MongoDB Server: you can access it on `localhost:27017` out-of-box, or
    via the k8s pod itself. You can change this in your k8s deployment file

1. To run the worker, `cd` into `worker-cities` and run `npm run dev`. It
will automatically begin populating `mongodb/uscities` with city information
from the US census and the Yahoo API.