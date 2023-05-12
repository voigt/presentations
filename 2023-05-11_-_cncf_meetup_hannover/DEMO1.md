
## prerequisites

* Nats-CLI tool  
```bash
brew tap nats-io/nats-tools  
brew install nats-io/nats-tools/nats
```
* wasmCloud CLI tool (wash) (link to [installation](https://wasmcloud.com/docs/installation/?os=mac))
```bash
brew tap wasmcloud/wasmcloud
brew install wash openssl@1.1
```

## Demo 1

### Create local wasmCloud Node

```bash
wash up
```

```bash
wash ctl get hosts
wash ctl get inventory <nodename>
```

### Deploy an example App

[Dogs and Cats](https://github.com/wasmCloud/examples/blob/main/ngs/dogs-and-cats/src/lib.rs)

**Start Actor:**

```bash
wash ctl start actor wasmcloud.azurecr.io/dogs-and-cats:0.2.4
```

**Start Provider:**

```bash
wash ctl start provider wasmcloud.azurecr.io/httpserver:0.17.0
wash ctl start provider wasmcloud.azurecr.io/httpclient:0.7.0
```

Link Actor to Provider:

```bash
wash ctl link put MCUCZ7KMLQBRRWAREIBQKTJ64MMQ5YKEGTCRGPPV47N4R72W2SU3EYMU VAG3QITQQ2ODAOWB5TTQSDJ53XK3SHBEIFNK4AYJ5RKAX2UNSCAPHA5M wasmcloud:httpserver ADDRESS=0.0.0.0:8081
wash ctl link put MCUCZ7KMLQBRRWAREIBQKTJ64MMQ5YKEGTCRGPPV47N4R72W2SU3EYMU VCCVLH4XWGI3SGARFNYKYT2A32SUYA2KVAIV2U2Q34DQA7WWJPFRKIKM wasmcloud:httpclient
```
