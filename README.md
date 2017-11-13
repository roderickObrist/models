# Installation Instructions

```
sudo apt-get install git gnupg-curl zlib1g-dev linux-headers-$(uname -r) openssh-server
git clone --recursive https://github.com/roderickObrist/models.git
cd models
````

# CUDA
```
# CUDA Headers for recomplication
sudo dpkg -i cuda-repo-ubuntu1604_9.0.176-1_amd64.deb
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub
sudo apt-get update
sudo apt-get install cuda

# Post installation environment variables
echo "export PATH=/usr/local/cuda-9.0/bin\${PATH:+:\${PATH}}" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}" >> ~/.bashrc
echo "export CUDA_HOME=/usr/local/cuda-9.0" >> ~/.bashrc
echo "export CUDA_VISIBLE_DEVICES=1" >> ~/.bashrc
sudo reboot

# Test CUDA
cat /proc/driver/nvidia/version

# Get CUDNN
sudo dpkg -i libcudnn7_7.0.3.11-1+cuda9.0_amd64.deb
sudo dpkg -i libcudnn7-dev_7.0.3.11-1+cuda9.0_amd64.deb
sudo dpkg -i libcudnn7-doc_7.0.3.11-1+cuda9.0_amd64.deb

# Test CUDNN
mkdir tmp
cp -r /usr/src/ tmp
cd tmp/mnistCUDNN
make clean && make && sudo ./mnistCUDNN

# look for Test passed!
cd ../../
rm -r tmp
```

*Syntaxnet Modified Instructions*
```
# Install Java for Bazel
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get -y install oracle-java8-installer libcupti-dev graphviz libgraphviz-dev swig python-pip gfortran

sudo dpkg -i bazel_0.7.0-linux-x86_64.deb

pip install --upgrade pip
pip install -U mock asciitree protobuf==3.3.0 autograd enum34 numpy
pip install pygraphviz \
  --install-option="--include-path=/usr/include/graphviz" \
  --install-option="--library-path=/usr/lib/graphviz/"

sudo -H python -m pip install 

# modify as per diff patch https://github.com/tensorflow/models/issues/2355
cp variant_op_registry.cc research/syntaxnet/tensorflow/tensorflow/core/framework/variant_op_registry.cc
# Modified instructions for GPU support https://github.com/tensorflow/models/issues/248
cp build_defs.bzl.tpl tensorflow/third_party/gpus/cuda/build_defs.bzl.tpl
cp CROSSTOOL_nvcc.tpl tensorflow/third_party/gpus/crosstool/CROSSTOOL

cd research/syntaxnet/tensorflow

# CUDNN is found here/usr/lib/x86_64-linux-gnu
./configure
```

```
export TF_NEED_CUDA=1
export CUDA_TOOLKIT_PATH=$CUDA_HOME
export TF_CUDA_VERSION=9.0
export TF_CUDNN_VERSION=7
export CUDNN_INSTALL_PATH=$CUDA_HOME

#simlink a library that does not get placed correctly
sudo ln -s /usr/lib/x86_64-linux-gnu/libcudnn.so.7 $CUDA_HOME/libcudnn.so.7

cd ..

bazel test -j1 \
  -c opt --config=cuda --define using_cuda_nvcc=true \
  --define using_gcudacc=true syntaxnet/... util/utf8/... \
  --action_env=PYTHON_BIN_PATH=/usr/bin/python \
  --action_env=PYTHON_LIB_PATH=/usr/local/lib/python2.7/dist-packages
```


