# Installation Instructions

```
# Basic Deps
sudo apt-get install git gnupg-curl zlib1g-dev linux-headers-$(uname -r) openssh-server
git clone --recursive https://github.com/roderickObrist/models.git
cd models

# CUDA
sudo dpkg -i cuda-repo-ubuntu1604_9.0.176-1_amd64.deb
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub
sudo apt-get update
sudo apt-get install cuda

# Post installation environment variables
cat env.txt >> ~/.bashrc

# important to reboot
sudo reboot

# Test CUDA
cat /proc/driver/nvidia/version

# Get CUDNN
sudo dpkg -i libcudnn7_7.0.3.11-1+cuda9.0_amd64.deb
sudo dpkg -i libcudnn7-dev_7.0.3.11-1+cuda9.0_amd64.deb
sudo dpkg -i libcudnn7-doc_7.0.3.11-1+cuda9.0_amd64.deb

# Test CUDNN
mkdir tmp
cp -r /usr/src/cudnn_samples_v7/mnistCUDNN tmp
cd tmp/mnistCUDNN
make clean && make && ./mnistCUDNN

# look for Test passed!
cd ../../
rm -r tmp

# Install Java for Bazel
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get -y install oracle-java8-installer libcupti-dev graphviz libgraphviz-dev swig python-pip gfortran

sudo dpkg -i bazel_0.7.0-linux-x86_64.deb

# Ignore warnings, do not upgrade pip
pip install mock
pip install asciitree
pip install protobuf==3.3.0
pip install autograd
pip install enum34
pip install numpy
pip install pygraphviz \
  --install-option="--include-path=/usr/include/graphviz" \
  --install-option="--library-path=/usr/lib/graphviz/"

# Set tensorflow last stable release
cd research/syntaxnet/tensorflow
git checkout e120e50bf5fd883cc28b584c720d959341db502d

# modify as per diff patch https://github.com/tensorflow/models/issues/2355
cp ../../../variant_op_registry.cc tensorflow/core/framework/variant_op_registry.cc

# Modified instructions for GPU support https://github.com/tensorflow/models/issues/248
cp ../../../build_defs.bzl.tpl third_party/gpus/cuda/build_defs.bzl.tpl
cp ../../../CROSSTOOL_nvcc.tpl third_party/gpus/crosstool/CROSSTOOL_nvcc.tpl

# CUDNN is found here/usr/lib/x86_64-linux-gnu
./configure

# Simlink a library that does not get placed correctly
sudo ln -s /usr/lib/x86_64-linux-gnu/libcudnn.so.7 $CUDA_HOME/libcudnn.so.7

cd ..

#flatbuffers is mising in syntaxmet
cp -r tensorflow/third_party/flatbuffers third_party/flatbuffers

bazel test \
  -c opt --config=cuda --define using_cuda_nvcc=true \
  --define using_gcudacc=true syntaxnet/... util/utf8/... \
  --action_env=PYTHON_BIN_PATH=/usr/bin/python \
  --action_env=PYTHON_LIB_PATH=/usr/local/lib/python2.7/dist-packages
```
