# helm chart: kanidm-k8s

A helm chart that deploys the [kanidm project](https://github.com/kanidm/kanidm) into any [kubernetes](https://kubernetes.io/) cluster.

Read the [kanidm docs/install the server](https://github.com/kanidm/kanidm/blob/master/kanidm_book/src/installing_the_server.md).

## Features

The helm chart provides

* persistent storage class
* secret management
* container security
* auto-scaling (hpa)

## Pre-requisites

* microK8s (or similar local k8s) installed for a local deployment
* an account with a cloud provider azure, gcp, aws, digital ocean

## Installation

clone repo:

```bash
$ git clone git@github.com:ronamosa/kanidm-k8s.git
$ cd kanidm-k8s/
```

### Local Setup: microK8s

Quick version:

```bash
$ sudo snap install microk8s --classic
$ microk8s.status
$ microk8s kubectl get all
```

Read [microK8s docs](https://microk8s.io/docs) for more details.

enable helm3

```bash
$ microk8s enable helm3
```

### Cloud Setup: Azure Kubernetes Service

_coming soon._

## KanIDM Setup

The kanidm server needs the following configs to boot up:

* `server.toml` config file
* persistent volume for the db file
* (optional) tls cert, key

### server.toml

This file is coming by way of `configMap` under `config.yaml`

```yaml
...
data:
  server.toml: |
    bindaddress = "0.0.0.0:8443"
    db_path = "/db/kanidm.db"
    tls_ca = "/ssl/ca.pem"
    tls_cert = "/ssl/cert.pem"
    tls_key = "/ssl/key.pem"
    log_level = "verbose"
...
```

### create tls certs, keys

Local: for a dev, local setup you can use self-signed certs.

A few things to keep in mind with this local setup & self-signed certs

* cert uses DNS in the SAN, over IP
* need to add the kanidm pods clusterIP entry to `/etc/hosts` to make this local DNS SAN work.
* you only get the kanidm pod clusterIP _after_ you deploy the chart.

Create your certs by running `./certs/insecure_generate_tls.sh`.

The default DNS alt_name will be `k8s.kanidm.local`, so once you have the kanidm pods clusterIP, this will become the entry in your `/etc/hosts` file e.g.

```bash
10.152.183.232  k8s.kanidm.local
```

### secrets.yaml

The `ca.pem`, `cert.pem` and `key.pem` files will be deployed as kubernetes secret objects by converting each file to a base64 hash and entering it into the `helm/templates/secrets.yaml` file:

```bash
$ base64 -w0 certs/ca.pem
$ base64 -w0 certs/cert.pem
$ base64 -w0 certs/key.pem
```

copy/paste that hashes into the appropriate data blocks in `secrets.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
 name: {{ .Release.Name }}-certs
type: Opaque
data:
 ca.pem: <base64_hash_goes_here>
 cert.pem: <base64_hash_goes_here>
 key.pem: <base64_hash_goes_here>
```

The volumes for the database and the tls certs will be setup by `deployment.yaml` as follows:

```yaml
# mount points
volumeMounts:
  - name: "config"
    mountPath: "/data"
  - name: "data"
    mountPath: "/db"
  - name: "certs"
    mountPath: "/ssl"

# volumes
volumes:
  - name: config
    configMap:
      name: {{ .Release.Name }}-config
  - name: certs
    secret:
      secretName: {{ .Release.Name }}-certs
  - name: data
    emptyDir: {}
```

## Deploy kanidm

### Local: microK8s

```bash
$ microk8s.helm3 install --debug kanidm helm/kanidm
```

get clusterIP address

```bash
$ microk8s.kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
kanidm       ClusterIP   10.152.183.232   <none>        8443/TCP,3389/TCP   32m
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP             23h
```

update `/etc/hosts` e.g.

```bash
10.152.183.232  k8s.kanidm.local
```

test kanidm setup _(note: you need `kanidm` binary installed first, see [kanidm docs](https://github.com/kanidm/kanidm/blob/master/kanidm_book/src/client_tools.md) for instructions)_

```bash
$ kanidm self whoami -C ca.pem -H https://hx0.kanidm.local:8443 --name anonymous
```

successful output looks like

```bash
name: anonymous
spn: anonymous@example.com
display: anonymous
uuid: 00000000-0000-0000-0000-ffffffffffff
groups: [Group { name: "anonymous", uuid: "00000000-0000-0000-0000-ffffffffffff" }]
claims: []
```

### Commands

```bash
$ something here
```

### Basic usage

```bash
$ microk8s.helm3 upgrade --install --debug kanidm helm/kanidm
```
