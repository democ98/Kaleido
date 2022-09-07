# Kaleido

## Prerequisites

* **Docker**

* **Intel SGX OOT 2.11.0 Driver or DCAP 1.36.2 Driver**

* **Intel SGX PSW**

* **Rust nightly-2020-10-25**

* **[SGX-enabled PBC library](https://github.com/tehsunnliu/pbc-sgx) - comes preinstalled with docker image**

* **[SGX-enabled GMP library](https://github.com/intel/sgx-gmp) - comes preinstalled with docker image**

NOTE: Please install sgx-gmp uder default directory i.e. `/usr/local/`

## Pulling a Pre-Built Docker Container

We assume that you have [correctly installed docker](https://docs.docker.com/get-docker/):

First, pull the docker container, the below command will download the `latest`:

```bash
docker pull cesslab/sgx-rust
```

## Running with Intel SGX Driver

By default Kaleido runs on port 8080, you can set the port to whatever you want by setting `KALEIDO_PORT` environment variable.
To map this TCP port in the container to the port on Docker host you can set `-p <DOCKER_HOST_PORT>:<KALEIDO_PORT>`. For example, if we want to map Container's port `8080` to our Docker host port `80` we can add `-p 80:8080`. Similiary, for remote attestation and keys sharing Kaleido requires another port to be exposed. By default in docker container this port is set to `8088`. To map this port we can add `-p 8088:8088`.

### To run the container with OOT SGX driver, run

```bash
docker run -v <PATH_TO_KALEIDO_ROOT_DIR>:/root/Kaleido -p 80:8080 -p 8088:8088 --device /dev/isgx -ti cesslab/sgx-rust
```

### To run the container with DCAP SGX driver

Check your `/dev/` directory for `/dev/sgx_enclave` and `/dev/sgx_provision`
or
`/dev/sgx/enclave` and `/dev/sgx/provision`
and replace `<YOUR_ENCLAVE_DIR>` and `<YOUR_PROVISION_DIR>` with the your directory respectively.

```bash
docker run -v <PATH_TO_KALEIDO_ROOT_DIR>:/root/Kaleido -p 80:8080 -p 8088:8088 --device <YOUR_ENCLAVE_DIR> --device <YOUR_PROVISION_DIR> -ti cesslab/sgx-rust
```

for example if the sgx driver is located in `/dev/sgx_enclave` and `/dev/sgx_provision` then run the following command

```bash
docker run -v <PATH_TO_KALEIDO_ROOT_DIR>:/root/Kaleido -p 80:8080 -p 8088:8088 --device /dev/sgx_enclave --device /dev/sgx_provision -ti cesslab/sgx-rust
```

### To run the container in simulation mode

For testing and development purpose

```bash
docker run --env SGX_MODE=SW -v <PATH_TO_KALEIDO_ROOT_DIR>:/root/Kaleido -p 80:8080 -p 8088:8088 -ti cesslab/sgx-rust
```

## Build the Source Code

### Install GMP

Follow the instructions at [SGX-enabled GMP library](https://github.com/intel/sgx-gmp) to install GMP library. We recommend you not to set the `--prefix` parameter while configuring the library. This will by default install the library uder `/usr/local/` which is the requirement of Kaleido.

### Install SGX-enabled PBC Library

Follow the instructions at [SGX-enabled PBC library](https://github.com/tehsunnliu/pbc-sgx) to install PBC library.

### Build Kaleido

Apply for Intel Remote Attestation API keys at [Intel IAS (EPID Attestation) Service](https://api.portal.trustedservices.intel.com/EPID-attestation). Make sure SPID is **linkable**

Set SPID and API key received from Intel

```bash
export IAS_SPID=<YOUR_SPID>
export IAS_API_KEY=<YOUR_PRIMARY_KEY_OR_SECONDARY_KEY>
```

First `cd` back to Kaleido root directory

```bash
cd ../..
```

then run the following command to build Kaleido. By default `make` builds in Hardware Mode `SGX_MODE=HW` to build in Software Mode uncomment `SGX_MODE=SW`.

```bash
make #SGX_MODE=SW
```

### Run Kaleido

Optionally, you can also set logging and debugging envrionment variable. To do so set the follwing

```bash
export RUST_LOG="debug"
export RUST_BACKTRACE=1
```

If you are running Kaleido in SGX(Hardware) mode you will have to start `AESM`, execute those commands in your terminal.
```bash
LD_LIBRARY_PATH=/opt/intel/sgx-aesm-service/aesm/
/opt/intel/sgx-aesm-service/aesm/aesm_service
```

Finally, To run Kaleido navigate to `/bin` and execute `app`

```bash
cd bin
./app
```

## Kaleido API Calls.

### `process_data`

**Description**: This function takes base64 encoded `data` for which **PoDR2** needs to be calculated. `block_size` and `segment_size` determines the size of each chunk of the `data` while calculating PoDR2. And the `callback_url` is the url where the computed PoDR2 result will be posted. 

**Request**
```bash
curl -H 'Content-Type: application/json' -X POST http://localhost/process_data -d '{"data":"aGk=", "block_size":10485, "segment_size":1, "callback_url":<REPLACE_WITH_CALLBACK_URL>}'
```

**Response**: The data will be posted back to the `callback_url` provided above with the following sample content.
```json
{
  "t": {
    "t0": {
      "name": "70FB321WFqzc9w67hcNF81rh2/b4T9lJKjy9YL8r8sA=",
      "n": 1,
      "u": [
        "QAK+f/glOhEIZfy16LX5K9n+pwE/sSg9+y/uNedJWq8B",
        "Pz7f+BOiRIUaRA4o3aQ7pUR61OKl5m4zyMPnXJ2L9VcB",
        "+5X5w9nbAWgkSj0zUE66aHVGSxDvKb/UPD/bwWmXFPQB"
      ]
    },
    "signature": "xNvKLcODuNqkEyPYqMK/+acPOQ+70SaSJP/nVnuEjHIA"
  },
  "sigmas": [
    "CDmNieOMKub3+DiFzssvnzOyXuaSjLhC1kUypab8dpkB"
  ],
  "pkey": "1IMbGs/VlFJ+x55igbsrPfWpONBAk+Dx4BqVnMMFL11WY2ROoraEESY2y9fHTrggvpHukH+wbSaTfbY+MinhRQA="
}
```
