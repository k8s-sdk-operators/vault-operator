# Vault Operator



## Operator SDK

##### This is the command used to create the original operator (don't do this again)
```
operator-sdk new vault-operator --api-version=vault.k8s-sdk-operators.lugs.org/v1alpha1 --kind=VaultService
```

##### Merge code from github/hasbro17/operator-sdk-samples/vault-operator

The only files changed after running the operator-sdk were
    pkg/apis/vault/alpha1/types.go
    pkg/stub/handler.go


### Build and push the app-operator image to a public registry such as quay.io

```
operator-sdk build quay.io/jkevlin/vault-operator

```

### Deploy Vault Operator on Minikube

##### Setup RBAC

```
sed -e 's/<namespace>/default/g' \
    -e 's/<service-account>/default/g' \
    deploy/rbac-template.yaml > deploy/rbac.yaml

kubectl -n default create -f deploy/rbac.yaml

```

### Deploying the etcd operator

The Vault operator employs the [etcd operator][etcd-operator] to deploy an etcd cluster as the storage backend.

1. Create the etcd operator Custom Resource Definitions (CRD):

    ```
    kubectl create -f deploy/etcd-crds.yaml

    ``` 
2. Deploy the etcd operator:

    ```sh
    kubectl -n default create -f deploy/etcd-operator-deploy.yaml

    ```


### Deploying the Vault operator

1. Create the Vault CRD:

    ```
    kubectl create -f deploy/vault-crd.yaml

    ```

2. Deploy the Vault operator:

    ```
    kubectl -n default create -f deploy/deployment.yaml

    ```

3. Verify that the operators are running:    

      ```
      $ kubectl -n default get deploy
      NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
      etcd-operator    1         1         1            1           5m
      vault-operator   1         1         1            1           5m
      ```

### Deploying a Vault cluster

A Vault cluster can be deployed by creating a `VaultService` Custom Resource(CR). For each Vault cluster the Vault operator will also create an etcd cluster for the storage backend.

1. Create a Vault CR that deploys a 2 node Vault cluster in high availablilty mode:

    ```
    kubectl -n default create -f deploy/vault-cr.yaml

    ```

2. Wait until the `example-...` pods for the etcd and Vault cluster are up:

    ```
    $ kubectl -n default get pods
    NAME                              READY     STATUS    RESTARTS   AGE
    etcd-operator-78899f87f6-qdn5h    3/3       Running   0          10m
    example-7678c8f49c-kfx2w          1/2       Running   0          2m
    example-7678c8f49c-pqrj8          1/2       Running   0          2m
    example-etcd-7lpjg7n76d           1/1       Running   0          2m
    example-etcd-dhxrksssgx           1/1       Running   0          2m
    example-etcd-s7mzhffz92           1/1       Running   0          2m
    vault-operator-5976f74f84-pxkf6   1/1       Running   0          10m
    ```

3. Get the Vault pods:

    ```
    $ kubectl -n default get pods -l app=vault,vault_cluster=example
    NAME                       READY     STATUS    RESTARTS   AGE
    example-7678c8f49c-kfx2w   1/2       Running   0          2m
    example-7678c8f49c-pqrj8   1/2       Running   0          2m
    ```

### Using the Vault cluster

See the [Vault usage guide](./doc/user/vault.md) on how to initialize, unseal, and use the deployed Vault cluster.

Consult the [monitoring guide](./doc/user/monitoring.md) on how to monitor and alert on a Vault cluster with Prometheus.

See the [recovery guide](./doc/user/recovery.md) on how to backup and restore Vault cluster data using the etcd opeartor

For an overview of the default TLS configuration or how to specify custom TLS assets for a Vault cluster see the [TLS setup guide](doc/user/tls_setup.md).

### Uninstalling Vault operator

1. Delete the Vault custom resource:

    ```
    kubectl -n default delete -f deploy/vault-cr.yaml
    ```

2. Delete the operators and other resources:

    ```
    kubectl -n default delete deploy vault-operator etcd-operator
    kubectl -n default delete -f deploy/rbac.yaml
    ```

[vault]: https://www.vaultproject.io/
[etcd-operator]: https://github.com/coreos/etcd-operator/




