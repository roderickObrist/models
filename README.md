# Installation Instructions

```
git clone --recursive git@github.com:roderickObrist/models.git
cd models
````

# CUDA
```
# CUDA Headers for recomplication
sudo apt-get install linux-headers-$(uname -r)

sudo dpkg -i cuda-repo-ubuntu1604_9.0.176-1_amd64.deb
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub
sudo apt-get update
sudo apt-get install cuda

# Post installation environment variables
echo "export PATH=/usr/local/cuda-9.0/bin\${PATH:+:\${PATH}}" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64 \${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}" >> ~/.bashrc
source ~/.bashrc

# Test CUDA
echo $PATH
echo $LD_LIBRARY_PATH
cat /proc/driver/nvidia/version

# Get CUDNN
sudo dpkg -i libcudnn7_7.0.3.11-1+cuda9.0_amd64.deb
sudo dpkg -i libcudnn7-dev_7.0.3.11-1+cuda9.0_amd64.deb
sudo dpkg -i libcudnn7-doc_7.0.3.11-1+cuda9.0_amd64.deb

echo "export CUDA_HOME=/usr/local/cuda-9.0" >> ~/.bashrc
source ~/.bashrc

# Test CUDNN
mkdir tmp
cp -r /usr/src/cudnn_samples_v7/ tmp
cd tmp/cudnn_samples_v7/mnistCUDNN
make clean && make
./mnistCUDNN
cd ../../../
rm temp

# LIBCUPTI
sudo apt-get install libcupti-dev

sudo reboot
```

*Syntaxnet Modified Instructions*
```
# Install Java for Bazel
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer

wget https://github.com/bazelbuild/bazel/releases/download/0.7.0/bazel_0.7.0-linux-x86_64.deb
sudo dpkg -i bazel_0.7.0-linux-x86_64.deb

sudo apt-get install swig
pip install -U protobuf==3.3.0
pip install mock
pip install asciitree
pip install numpy
sudo apt-get install -y graphviz libgraphviz-dev
pip install pygraphviz --install-option="--include-path=/usr/include/graphviz" --install-option="--library-path=/usr/lib/graphviz/"

sudo apt install gfortran
sudo -H python -m pip install autograd
sudo -H pip install enum34

# modify as per diff patch https://github.com/tensorflow/models/issues/2355
cp variant_op_registry.cc models/research/syntaxnet/tensorflow/tensorflow/core/framework/variant_op_registry.cc

cd research/syntaxnet/tensorflow

# CUDNN is found here/usr/lib/x86_64-linux-gnu
./configure
```

# Modified instructions for GPU support https://github.com/tensorflow/models/issues/248
```
# In the file tensorflow/third_party/gpus/crosstool/CROSSTOOL,
# replace every `cxx_builtin_include_directory: "%{cuda_include_path}"`
# with `cxx_builtin_include_directory: "your/cuda/home/path/include"`

# Force Tensorflow to use Cuda by changing the //conditions:default part in syntaxnet/syntaxnet.bzl from `if_false` to `if_true`.

# Do the same thing for tensorflow/third_party/gpus/cuda/build_defs.bzl
export TF_NEED_CUDA=1
export CUDA_TOOLKIT_PATH=$CUDA_HOME
export TF_CUDA_VERSION=9.0
export TF_CUDNN_VERSION=7.0
export CUDNN_INSTALL_PATH=$CUDA_HOME
```

```
cd ..

bazel test \
  -c opt --config=cuda --define using_cuda_nvcc=true \
  --define using_gcudacc=true syntaxnet/... util/utf8/... \
  --action_env=PYTHON_BIN_PATH=/usr/bin/python \
  --action_env=PYTHON_LIB_PATH=/usr/local/lib/python2.7/dist-packages
```


