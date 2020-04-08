# OpenVINO

This directory contains a Makefile and a template manifest for 
OpenVINO toolkit with FPGA plugin (as of this writing, version 2019_R3.1).
We use the "classifier_sample" example from the OpenVINO distribution as a concrete application running
under Graphene-SGX. We test OpenVINO with offload to the FPGA. (Plugin: Hetero:FPGA, CPU) i

This was tested on a machine with SGX and Ubuntu 16.04.

# Prerequisites

1. Accelerator Stack RTE installed in the default install directory 

2. OpenVINO R3.1 installed in the default directory

3. Copy the following :
    classification_sample_sync copied to ./
    Copy model to be tested to ./models/ 
        For e.g. squeenzenet or Resnet 50. Refer to the Model zoo and Optimizer guide on the OpenVINO website
    Copy image to be tested to ./pics/ 
    Copy Shared library dependencies to ./lib 
        Refer to the manifest section: 'sgx.trusted_files.*' for information on what all shared libraries need to be copied.

4. Configuration changes needed to Manifest or Makefile are marked with TODO:
    Change the DEVICE_PATH
    
# Quick Start

```sh
# Build OpenVINO-FPGA with graphene-SGX;
make SGX=1

# run OpenVINO/benchmark_app in non-SGX Graphene

   ./classification_sample_sync -d HETERO:FPGA,CPU -i ./pics/car.png -m ./models/squeezenet1.1.xml -api sync -niter 1 -nireq 1 -nstreams 1

# run OpenVINO/benchmark_app in Graphene-SGX

    SGX=1 ./openvino_fpga.manifest.sgx -d HETERO:FPGA,CPU -i ./pics/car.png -m ./models/squeezenet1.1.xml 

Refer to the online OpenVINO document on benchmark_app regarding more information on output and configurable parameters

```
