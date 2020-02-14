# orchesto-csi in k8s

## Description

#####⚠️ This is a preview of orchesto-csi not meant for production use

This guide contains instructions on how to deploy orchesto-csi to your kubernetes cluster.

When you are done you will have a pod running in your cluster using Orchesto as csi backend.


## Preparation

Clone our orchesto-csi repo [orchesto-csi](http://github.com/zebware/orchesto-csi.git)

Update the configmap containing license.json in `orchesto/orchesto.yaml` with your license key.

Create a new namespace in your kubernetes cluster

```sh
kubectl create ns orchesto
```

## Postgres

Deploy Postgres

```sh
kubectl apply -f orchesto/postgres.yaml
```

## Orchesto

Run deployment

```sh
kubectl apply -f orchesto/orchesto.yaml
```

Configure port-forwarding

```sh
$ kubectl -n orchesto port-forward deployments/orchesto-gateway 9090:9090
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090
```

Visit your [orchesto gateway](http://localhost:9090/) through the browser. Rember to store your Access Key and Secret Access Key. Click continue without Vault.

Configure Orchesto through the ui

1. Add backend

If you forgot to store your access key and secret key you can retrieve them like so:

```sh
$ kubectl -n orchesto logs deployment/orchesto-gateway|grep 'Username:' -A 2
  Username:            admin
  Access Key:          AIKAVL8J0KTHK95ASIUN
  Secret Key/Password: EQF8fUOO3jaZMnk5vxxotVIrMt5XHibaCk50OHX5
```

## CSI driver

Go ahead and deploy our csi-driver to your cluster

provisioner:

```sh
kubectl apply -f csi/provisioner.yaml
```

attacher:

```sh
kubectl apply -f csi/attacher.yaml
```

Make sure the attacher and provisioner is up and ready before continuing

deploy csi-s3

```sh
kubectl apply -f csi/csi-s3.yaml
```

and a storage class

```sh
kubectl apply -f csi/storageclass.yaml
```


## Persistent Volume Claim

Creating a pvc requires some editing of `pvc/pvc.yaml`. Edit the fields `accessKeyID`, `secretAccessKey` and `region` with the correct values. The `region` value must match the region of your backend that was configured in the step Orchesto, above. The `region` must match the region of your backend that was configured in Orchesto.

You can find the region in the [dashboard](http://localhost:9090/orchesto/#/dashboard)

When the `pvc.yaml` file is updated go ahead and apply it.

```sh
kubectl apply -f pvc/pvc.yaml
```


## pod

Launch a pod

```sh
kubectl apply -f pod/pod.yaml
```

## Test

Create a secret file

```sh
$ kubectl exec -n orchesto -it orchesto-pod -- bash
root@orchesto-pod:/# echo "My very secret secret stored in the cloud" > /var/lib/www/html/secret.txt
```

1. Visit your backend CSP ☁️ and look at the files in the bucket you created.
2. Visit your [orchesto gateway bucket](http://localhost:9090/orchesto/#/browse/) and download your file and open it