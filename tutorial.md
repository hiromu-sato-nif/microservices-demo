# Deploying to an Istio-enabled cluster 
## Overview
This repository provides an [`istio-manifests`](/istio-manifests) directory containing ingress resources (an Istio `Gateway` and `VirtualService`) needed to expose the app frontend running inside a Kubernetes cluster.

You can apply these resources to your cluster in addition to the `kuberentes-manifests`, then use the Istio IngressGateway's external IP to view the app frontend. See the following instructions for Istio steps.   

## Select Project

<walkthrough-project-setup>
</walkthrough-project-setup>

<walkthrough-watcher-constant key="zone" value="us-central1-b">
</walkthrough-watcher-constant>

## Steps
 
1. Create a GKE cluster with at least 4 nodes, machine type `e2-standard-4`. 

    ```bash
    gcloud container clusters create onlineboutique \
        --project={{project-id}} --zone={{zone}} \
        --machine-type=e2-standard-4 --num-nodes=4
    ```

2. Download and extract the latest release automatically (Linux or macOS):

    ```bash
    curl -L https://istio.io/downloadIstio | sh -
    ```

3. Move to the Istio package directory. For example, if the package is `istio-1.7.4`

    ```bash
    cd istio-1.7.4
    ```

4. Add the istioctl client to your path (Linux or macOS)

    ```bash
    export PATH=$PWD/bin:$PATH; cd ..
    ```

5. For this installation, we use the `demo` configuration profile. It’s selected to have a good set of defaults for testing, but there are other profiles for production or performance testing.

    ```bash
    istioctl install --set profile=demo
    ```


3. Enable Istio sidecar proxy injection in the `default` Kubernetes namespace. 

   ```bash
   kubectl label namespace default istio-injection=enabled
   ```

4. Apply all the manifests in the `/release` directory. This includes the Istio and Kubernetes manifests. 

   ```sh
   kubectl apply -f ./release 
   ```

5. See pods are in a healthy and ready state.

    ```bash
    kubectl get pods
    ```

6. Find the IP address of your Istio gateway Ingress or Service, and visit the
   application frontend in a web browser.

   ```sh
   INGRESS_HOST="$(kubectl -n istio-system get service istio-ingressgateway \
      -o jsonpath='{.status.loadBalancer.ingress[0].ip}')"
   echo "$INGRESS_HOST"
   ```

   ```sh
   curl -v "http://$INGRESS_HOST"
   ```