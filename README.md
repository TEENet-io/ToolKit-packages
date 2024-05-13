# ToolKit-packages

## Prerequisites
Rent a cloud server that supports TEE hardware.   
TEENet's ToolKit currently supports Intel SGX environment.  
#### Alibaba Cloud  
Only ***c7t / g7t / r7t*** instances support Intel SGX. For more information, check: https://help.aliyun.com/zh/ecs/user-guide/build-an-sgx-encrypted-computing-environment#53b0d34b7cnlu   
#### IBM Cloud
Only ***Balanced-bx3d*** and ***Compute-cx3d*** virtual servers support Intel SGX. For more information, check: https://cloud.ibm.com/docs/vpc?topic=vpc-about-sgx-vpc  
#### Tencent Cloud
Only ***M6ce*** instance supports Intel SGX. For more information, check: https://www.tencentcloud.com/zh/document/product/213/45510   

## Install ToolKit
### Install via docker
- Git clone this repo  
  ```shell
  git clone https://github.com/TEENet-io/ToolKit-packages.git
  ```
- Build image
  ```shell
  cd ToolKit-packages

  docker build -t teenet-toolkit:latest -f Dockerfile.TEENetToolKit .
  ```
## Build & Run Program inside TEE via TEENet ToolKit
Take mapprotocol/compass as an example:   
### Adaption
- Run docker image as container
  ```shell
  docker run -it --device /dev/sgx_enclave:/dev/sgx_enclave --device /dev/sgx_provision:/dev/sgx_provision teenet-toolkit:latest
  ```
- Git clone mapprotocol/compass
  ```shell
  git clone https://github.com/mapprotocol/compass.git
  ```
- Inject mocked C code where the project itself or referenced package contains calls of C
  ```shell
  cd compass/connections/ethereum

  vim c.go
  ```

  Inject these lines into c.go
  ```golang
  
  package ethereum
  
  /*
  int __pthread_register_cancel() {return 0;}
  int __pthread_unregister_cancel() {return 0;}
  int __sigsetjmp() {return 0;}
  */
  import "C"
  
  ```
### Build & Run
- Build project
  ```shell
  cd compass/cmd/compass

  CGO_CFLAGS=-D_FORTIFY_SOURCE=0  teenettoolkit-go build -o ../../build/compass-oracle
  cp ../../eth2/eth2-proof ../../build/
  ```
- Configure enclave
  ```shell
  # Enclave configuration file should be at the same path with compiled executable target, like compass-oracle
  cd compass/build

  vim enclave.json
  ```
  Reference configure.example.json in this repo to write your enclave.json file
- Run
  ```shell
  # Sign the executable target first
  cd compass/build

  teenettoolkit sign compass-oracle

  teenettoolkit run compass-oracle maintainer --blockstore ../block-eth-map --config ../config.json
  ``` 

