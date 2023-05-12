## Demo 2

### Create K8S Cluster

**Prerequisites**

- Kubernetes Cluster (KIND)
- NATS (NGS)
- wash cli

Create Cluster

```bash
kind create cluster --name wasmcloud
Creating cluster "wasmcloud" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-wasmcloud"
You can now use your cluster with:

kubectl cluster-info --context kind-wasmcloud

Thanks for using kind! 😊
```

### Create NATS

In this demo we will use NGS (NATS Global Service) as a hosted NATS service. You can also use any other NATS (cluster).

Set Credentials:

```bash
CREDS_FILE=".local/share/nats/nsc/keys/creds/synadia/quizzical_easley/default.creds"
```

Check if NATS available

```
nats req ngs.echo 'Anyone out there?'
```

### Deploy Helm Chart

[Helm-Chart](https://github.com/wasmCloud/wasmcloud-otp/blob/main/wasmcloud_host/chart/values.yaml)

```bash
helm repo add wasmcloud https://wasmcloud.github.io/wasmcloud-otp/
```

```bash
helm install --wait -f values.yaml \
  --set-file "nats.leafnode.credentials=${HOME}/${CREDS_FILE}" \
  --set-file wasmcloud.config.clusterSeed=clusterseed.nk wasm-meetup wasmcloud/wasmcloud-host
```

Expose Port 4000
```bash
kubectl port-forward deployment/wasm-meetup-wasmcloud-host 4001:4000
kubectl port-forward deployment/wasm-meetup-wasmcloud-host 4223:4222
```

### Create Local wasmtime instance

```bash
HOST_machine=second wash up \
  --nats-remote-url tls://connect.ngs.global \
  --nats-credsfile ~/${CREDS_FILE} \
  --cluster-seed $(cat clusterseed.nk) \
  --wasmcloud-js-domain core \
  --nats-js-domain extender
```

### Deploy Dogs and Cats (again)

**Start Actor:**

```bash
wash ctl start actor wasmcloud.azurecr.io/dogs-and-cats:0.2.4 --constraint kubernetes="true"
```

**Start Provider:**

```bash
wash ctl start provider wasmcloud.azurecr.io/httpserver:0.17.0 --constraint kubernetes="true"
wash ctl start provider wasmcloud.azurecr.io/httpclient:0.7.0 --constraint kubernetes="true"
```

Link Actor to Provider:

```bash
wash ctl link put MCUCZ7KMLQBRRWAREIBQKTJ64MMQ5YKEGTCRGPPV47N4R72W2SU3EYMU VAG3QITQQ2ODAOWB5TTQSDJ53XK3SHBEIFNK4AYJ5RKAX2UNSCAPHA5M wasmcloud:httpserver ADDRESS=0.0.0.0:8081
wash ctl link put MCUCZ7KMLQBRRWAREIBQKTJ64MMQ5YKEGTCRGPPV47N4R72W2SU3EYMU VCCVLH4XWGI3SGARFNYKYT2A32SUYA2KVAIV2U2Q34DQA7WWJPFRKIKM wasmcloud:httpclient
```

Add some magic:

```bash
wash ctl start provider wasmcloud.azurecr.io/applier:0.3.0 --constraint kubernetes="true"
wash ctl start provider wasmcloud.azurecr.io/nats_messaging:0.15.0 --constraint kubernetes="true"
wash ctl start actor wasmcloud.azurecr.io/service_applier:0.3.0 --constraint kubernetes="true"
```

## Service-Applier -> NATs
```bash
wash ctl link put \
MCF7GSDHIC6DMOXAJKD747RIRS7I54H7OIDEWGA5JMZJF3WNH6KNIKO3 \
VADNMSIML2XGO2X4TPIONTIC55R2UUQGPPDZPAVSC2QD7E76CR77SPW7 \
wasmcloud:messaging 'SUBSCRIPTION=wasmbus.evt.default,URI=nats://localhost:4222'
```

## Service-Applier -> Applier-Provider

```bash
wash ctl link put \
MCF7GSDHIC6DMOXAJKD747RIRS7I54H7OIDEWGA5JMZJF3WNH6KNIKO3 \
VDW26HWKJZMKRNIRABM3BEF7BP23XECL7CO4JAAUZ5YEB6UGQFVK7ANR \
cosmonic:kubernetes_applier
```

## Add DogAndCat Actor -> Server Link

```bash
wash ctl link put \
MCUCZ7KMLQBRRWAREIBQKTJ64MMQ5YKEGTCRGPPV47N4R72W2SU3EYMU \
VAG3QITQQ2ODAOWB5TTQSDJ53XK3SHBEIFNK4AYJ5RKAX2UNSCAPHA5M \
wasmcloud:httpserver ADDRESS=0.0.0.0:8081
```

[List of Capability Providers](https://github.com/wasmCloud/capability-providers#first-party-capability-providers)
