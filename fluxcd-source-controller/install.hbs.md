# Install Flux CD Source Controller

This topic tells you how to install Flux CD Source Controller from the Tanzu Application Platform (commonly known as TAP) package repository.

>**Note** Follow the steps in this topic if you do not want to use a profile to install Flux CD Source Controller.
For more information about profiles, see [Components and installation profiles](../about-package-profiles.hbs.md).

## <a id='sc-prereqs'></a>Prerequisites

Before installing Flux CD Source Controller:

- Complete all prerequisites to install Tanzu Application Platform. For more information, see [Prerequisites](../prerequisites.md).
- Install cert-manager on the cluster. For more information, see [Install cert-manager](../cert-manager/install.hbs.md).

## <a id="Configuration"></a> Configuration

The Flux CD Source Controller package has no configuration values.

##  <a id="installation"></a> Installation

To install Flux CD Source Controller from the Tanzu Application Platform package repository:

1. List version information for the package by running:

    ```console
    tanzu package available list fluxcd-source-controller.tanzu.vmware.com -n tap-install
    ```

    For example:

    ```console
    $ tanzu package available list fluxcd-source-controller.tanzu.vmware.com -n tap-install
        \ Retrieving package versions for fluxcd-source-controller.tanzu.vmware.com...
          NAME                                       VERSION  RELEASED-AT
          fluxcd-source-controller.tanzu.vmware.com  1.1.2    2024-03-11 00:00:00 -0500 -05
    ```

2. Install the package by running:

    ```console
    tanzu package install fluxcd-source-controller -p fluxcd-source-controller.tanzu.vmware.com -v VERSION-NUMBER -n tap-install
    ```

    Where:

    - `VERSION-NUMBER` is the version of the package listed in step 1.

    For example:

    ```console
    tanzu package install fluxcd-source-controller -p fluxcd-source-controller.tanzu.vmware.com -v 1.1.2 -n tap-install
    \ Installing package 'fluxcd-source-controller.tanzu.vmware.com'
    | Getting package metadata for 'fluxcd-source-controller.tanzu.vmware.com'
    | Creating service account 'fluxcd-source-controller-tap-install-sa'
    | Creating cluster admin role 'fluxcd-source-controller-tap-install-cluster-role'
    | Creating cluster role binding 'fluxcd-source-controller-tap-install-cluster-rolebinding'
    | Creating package resource
    - Waiting for 'PackageInstall' reconciliation for 'fluxcd-source-controller'
    | 'PackageInstall' resource install status: Reconciling

      Added installed package 'fluxcd-source-controller'
    ```

    This package creates a new namespace called `flux-system`. This namespace hosts all the elements of fluxcd.

3. Verify the package install by running:

    ```console
    tanzu package installed get fluxcd-source-controller -n tap-install
    ```

    For example:

    ```console
    tanzu package installed get fluxcd-source-controller -n tap-install
    \ Retrieving installation details for fluxcd-source-controller...
    NAME:                    fluxcd-source-controller
    PACKAGE-NAME:            fluxcd-source-controller.tanzu.vmware.com
    PACKAGE-VERSION:         0.16.0
    STATUS:                  Reconcile succeeded
    CONDITIONS:              [{ReconcileSucceeded True  }]
    USEFUL-ERROR-MESSAGE:
    ```

    Verify that `STATUS` is `Reconcile succeeded`.

    ```console
    kubectl get pods -n flux-system
    ```

    For example:

    ```console
    $ kubectl get pods -n flux-system
    NAME                                 READY   STATUS    RESTARTS   AGE
    source-controller-69859f545d-ll8fj   1/1     Running   0          3m38s
    ```

    Verify that `STATUS` is `Running`.


##  <a id="try-fluxcd-source-controller"></a> Try fluxcd-source-controller

1. Verify the main components of `fluxcd-source-controller` were installed by running:

    ```console
    kubectl get all -n flux-system
    ```

    Expect to see the following outputs or similar:

    ```console
    NAME                                     READY   STATUS    RESTARTS   AGE
    pod/source-controller-7684c85659-2zfxb   1/1     Running   0          40m

    NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    service/source-controller   ClusterIP   10.108.138.74   <none>        80/TCP    40m

    NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/source-controller   1/1     1            1           40m

    NAME                                           DESIRED   CURRENT   READY   AGE
    replicaset.apps/source-controller-7684c85659   1         1         1       40m
    ```

2. Verify all the CRD were installedby running:

    ```console
    kubectl get crds -n flux-system | grep ".fluxcd.io"
    buckets.source.toolkit.fluxcd.io                         2022-03-07T19:20:14Z
    gitrepositories.source.toolkit.fluxcd.io                 2022-03-07T19:20:14Z
    helmcharts.source.toolkit.fluxcd.io                      2022-03-07T19:20:14Z
    helmrepositories.source.toolkit.fluxcd.io                2022-03-07T19:20:14Z
    ```

    >**Note** You will communicate with `fluxcd-source-controller` through its CRDs.

3. If you are using a Git repository with a custom CA certificate, provide this certificate to the
   Flux CD Source Controller directly by including the CA in the service account used by the supply
   chain.

   The Tanzu Application Platform distribution of Flux CD Source Controller does not support the
   Tanzu Application Platform `shared.ca_cert_data` field. For more information about setting the CA
   in the service account, see
   [Use Git authentication with Supply Chain Choreographer](../scc/git-auth.hbs.md).

4. Follow these steps to consume a `GitRepository` object:

    1. Create the following `gitrepository-sample.yaml` file:

        ```console
        apiVersion: source.toolkit.fluxcd.io/v1
        kind: GitRepository
        metadata:
          name: gitrepository-sample
        spec:
          interval: 1m
          url: https://github.com/vmware-tanzu/application-accelerator-samples
          ref:
            branch: main
        ```

    2. Apply the created conf:

        ```console
        kubectl apply -f gitrepository-sample.yaml
        gitrepository.source.toolkit.fluxcd.io/gitrepository-sample created
        ```

    3. Verify the git-repository was fetched correctly:

        ```console
        kubectl get GitRepository
        NAME                   URL                                                               READY   STATUS                                                              AGE
        gitrepository-sample   https://github.com/vmware-tanzu/application-accelerator-samples   True    Fetched revision: main/132f4e719209eb10b9485302f8593fc0e680f4fc   4s
        ```

    For more examples, see the samples directory on [fluxcd/source-controller/samples](https://github.com/fluxcd/source-controller/tree/main/config/samples) in GitHub.

## <a id='resource-limits'></a>Configure resource limits

Each system has its unique resource requirements, and the default resources
provided by Flux Source Controller might not meet the needs of your specific use case.

Flux Source Controller uses the following resource limits by default:

```yaml
resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 50m
    memory: 64Mi
```

You can configure the resource limits by following these steps:

1. Create a `Secret` with the following `ytt` overlay:

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: fluxcd-source-overlay-secret
      namespace: tap-install
    stringData:
      fluxcd-source-overlay.yaml: |
          #@ load("@ytt:data", "data")
          #@ load("@ytt:overlay", "overlay")
          #@overlay/match by=overlay.subset({"kind":"Deployment", "metadata":{"name":"fluxcd-source-controller", "namespace": "flux-system"}}), expects="1+"
          ---
          spec:
            template:
              spec:
                containers:
                #@overlay/match by="name"
                - name: manager
                  resources:
                    limits:
                      cpu: "4"
                      memory: "4Gi"
                    requests:
                      cpu: "2"
                      memory: "2Gi"
    ```

1. Update the Tanzu Application Platform values file by including a `package_overlays` field:

    ```yaml
    package_overlays:
    - name: fluxcd-source-controller
      secrets:
      - name: fluxcd-source-overlay-secret
    ```

1. Update Tanzu Application Platform by running:

    ```console
    tanzu package installed update tap -p tap.tanzu.vmware.com -v 1.9.0 --values-file tap-values.yaml -n tap-install
    ```

For more information about the package customization, see [Customize your package installation](../customize-package-installation.hbs.md).

##  <a id="documentation"></a> Documentation

For documentation specific to fluxcd-source-controller, see the main repository
[fluxcd/source-controller](https://github.com/fluxcd/source-controller) in GitHub.
