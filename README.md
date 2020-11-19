# Introduction

This repo contains the minimal set of k8s artifacts necessary to setup 2 MinIO tenants to test
- [bucket replication](https://docs.minio.io/docs/minio-bucket-replication-guide.html)
- [ILM](https://docs.minio.io/docs/minio-bucket-lifecycle-guide.html)

## Prerequisites
- [kind](https://kind.sigs.k8s.io/)

## Setup

### Installing kind
The following command installs a kind based k8s cluster with 4 worker nodes.
```sh
kind create cluster --config=kind-simple.yaml --name op-dev
```

### Install MinIO k8s operator
This command installs the MinIO operator. To move to a more recent version of
the operator, modify `newTag` in the `kustomization.yaml` file.

```sh
kustomize build | kubectl apply -f -
```


### Deploy MinIO tenants
(Note: At the time of writing this, the tenants don't have TLS setup.)
1. Deploy tenant-1
```sh
kubectl create -f tenant-1.yaml
```

2. Deploy tenant-2
```sh
kubectl create -f tenant-2.yaml
```

### Setup bucket replication between tenant-1 and tenant-2
### Configuring MinIO tenants

1. Configuring tenant-2 (destination side)
```sh
kubectl run mc-t2 -n tenant-2 --generator=run-pod/v1 --env="MC_HOST_minio=http://minio:minio123@minio/" --rm -i --tty --image minio/mc:RELEASE.2020-05-28T23-43-36Z-157-ga4791cd3 --command -- /bin/sh
```

Create the destination bucket, say `dest`

```sh
mc mb minio/dest
```

2. Configuring tenant-1 (source side)

```sh
kubectl run mc-t1 -n tenant-1 --generator=run-pod/v1 --env="MC_HOST_minio=http://minio:minio123@minio/" --rm -i --tty --image minio/mc:RELEASE.2020-05-28T23-43-36Z-157-ga4791cd3 --command -- /bin/sh
```

Create the source bucket, say `source`
```sh
mc mb minio/source
```

3. Configure tenant-1's remote bucket target
(From the tenant-1's mc container)
```sh
mc admin bucket remote add minio/source http://minio:minio123@minio.tenant-2.svc.cluster.local/dest --service replication"
```
