# ToolKit-packages

## Prerequisites
### Rent a cloud server that supports TEE hardware.   
TEENet's ToolKit currently supports Intel SGX environment.  
- Alibaba Cloud  
  ***c7t / g7t / r7t*** instances support Intel SGX. For more information, check: https://help.aliyun.com/zh/ecs/user-guide/build-an-sgx-encrypted-computing-environment#53b0d34b7cnlu
  ***c7t***: Indonesia / Shenzhen, China / Shanghai, China / Hangzhou, China / Beijing, China  
  ***g7t***: Indonesia / Singapore / HongKong, China / Shenzhen, China / Shanghai, China / Hangzhou, China / Beijing, China
  ***r7t***: South Korea / Shenzhen, China / Shanghai, China / Hangzhou, China / Beijing, China    
- IBM Cloud  
  ***Balanced-bx3d*** and ***Compute-cx3d*** virtual servers support Intel SGX. For more information, check: https://cloud.ibm.com/docs/vpc?topic=vpc-about-sgx-vpc  
- Tencent Cloud  
  ***M6ce*** instance supports Intel SGX. For more information, check: https://www.tencentcloud.com/zh/document/product/213/45510
- Microsoft Azure   
 ***DCsv2-series*** and ***DCsv3/DCdsv3-series*** support Intel SGX. For more information, check: https://learn.microsoft.com/en-us/azure/confidential-computing/quick-create-portal   
### Check SGX hardware status
Open terminal to check whether SGX is enabled or not:
```shell
sudo apt install cpuid

cpuid | grep SGX
```
If SGX is enabled in BIOS, you may see output contains info like this:
```shell
    SGX: Software Guard Extensions supported = true
    SGX_LC: SGX launch config supported      = true
    SGX capability (0x12/0):
      SGX1 supported                         = true
```
- SGX: Software Guard Extensions supported is true if the hardware supports it.
- SGX_LC: SGX launch config supported is true if the hardware also supports FLC. This is required for attestation.
- SGX1 supported is true if it's enabled in the BIOS.
### Check driver
```shell
ls /dev/*sgx*
```
If your SGX environment is good to go, you may see output contains info like this:
```shell
/dev/sgx:
enclave  provision
```

## Install ToolKit
### Install via docker
- Git clone this repo  
  ```shell
  git clone https://github.com/TEENet-io/ToolKit-packages.git
  ```
- Build image
  ```shell
  cd ToolKit-packages

  docker build -t teenet-toolkit:latest -f Dockerfile.ToolKit .
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
  Reference enclave.example.json in this repo to write your enclave.json file
- Run  
  ```shell
  cd compass/build

  # Executable must be signed before they can be run in a SGX enclave.
  # The purpose is to ensure the integrity and consistency of the program. 
  # teenettoolkit sign <Executable> will generates a new key "private.pem" and search configuration "enclave.json" in the current directory and signs the executable.
  teenettoolkit sign compass-oracle

  cd ..

  # You can configure your own config.json and key.json
  teenettoolkit run ./build/compass-oracle maintainer --blockstore ./block-eth-map --config ./config.json --keystorePath ./key.json
  ```
- Re-Run   
  If your server restarts or the code is updated, you can recompile the program and run it according to the above process.

### Troubleshooting
- Failed to sign executable with ```teenettoolkit sign```
  ```shell
  # Error may occur like following
  
  TEENetToolkit v0.1.0 (5869b4e11e4f0868275d2bf03803736bcc3a03da)
  Generating new private.pem
  ERROR: /opt/teenettoolkit/bin/teenettoolkit-oesign ERROR: oe_sgx_build_enclave(): result=OE_FAILURE (0x1)
  2024-05-17T19:06:49+0800.946085Z [(H)ERROR] tid(0x7f6894761740) | symbol not found: __isoc23_strtoul [/ertbuild/3rdparty/openenclave/openenclave-src/host/sgx/loadelf.c:_link_elf_image:848]
  2024-05-17T19:06:49+0800.954790Z [(H)ERROR] tid(0x7f6894761740) | symbol not found: __isoc23_fscanf [/ertbuild/3rdparty/openenclave/openenclave-src/host/sgx/loadelf.c:_link_elf_image:848]
  2024-05-17T19:06:49+0800.957557Z [(H)ERROR] tid(0x7f6894761740) | symbol not found: __isoc23_sscanf [/ertbuild/3rdparty/openenclave/openenclave-src/host/sgx/loadelf.c:_link_elf_image:848]
  2024-05-17T19:06:50+0800.029593Z [(H)ERROR] tid(0x7f6894761740) | symbol not found: __isoc23_strtol [/ertbuild/3rdparty/openenclave/openenclave-src/host/sgx/loadelf.c:_link_elf_image:848]
  2024-05-17T19:06:50+0800.092817Z [(H)ERROR] tid(0x7f6894761740) | 4 symbols not found (oe_result_t=OE_UNSUPPORTED_ENCLAVE_IMAGE) [/ertbuild/3rdparty/openenclave/openenclave-src/host/sgx/loadelf.c:_link_elf_image:866]
  2024-05-17T19:06:50+0800.092821Z [(H)ERROR] tid(0x7f6894761740) | :OE_UNSUPPORTED_ENCLAVE_IMAGE [/ertbuild/3rdparty/openenclave/openenclave-src/host/sgx/loadelf.c:_patch_relocations:1219]
  2024-05-17T19:06:50+0800.092823Z [(H)ERROR] tid(0x7f6894761740) | :OE_UNSUPPORTED_ENCLAVE_IMAGE [/ertbuild/3rdparty/openenclave/openenclave-src/host/sgx/loadelf.c:oe_load_elf_enclave_image:1550]
  2024-05-17T19:06:50+0800.093861Z [(H)ERROR] tid(0x7f6894761740) | :OE_UNSUPPORTED_ENCLAVE_IMAGE [/ertbuild/3rdparty/openenclave/openenclave-src/host/sgx/load.c:oe_load_enclave_image:66]
  2024-05-17T19:06:50+0800.093864Z [(H)ERROR] tid(0x7f6894761740) | :OE_FAILURE [/ertbuild/3rdparty/openenclave/openenclave-src/host/sgx/create.c:oe_sgx_build_enclave:996]

  ```

  This is related to C/C++ function called by Golang, try to fix it by injecting those missing symbols in ```c.go```.

  ```shell
  # In c.go
  package ethereum
  
  /*
  int __pthread_register_cancel() {return 0;}
  int __pthread_unregister_cancel() {return 0;}
  int __sigsetjmp() {return 0;}
  int __isoc23_strtoul() {return 0;}
  int __isoc23_fscanf() {return 0;}
  int __isoc23_sscanf() {return 0;}
  int __isoc23_strtol() {return 0;}
  */
  import "C"

  ```
  
### 
  
