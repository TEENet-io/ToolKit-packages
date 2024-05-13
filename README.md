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

## Adaption
```shell
git clone https://github.com/mapprotocol/compass.git

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

## Build & Run

