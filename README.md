# Installation Instructions

*Start with cuda*
http://docs.nvidia.com/cuda/cuda-installation-guide-linux/#choose-installation-method

```
# CUDA Headers for recomplication
sudo apt-get install linux-headers-$(uname -r)

# DPKG https://developer.nvidia.com/cuda-downloads
sudo dpkg -i cuda-repo-ubuntu1704_9.0.176-1_amd64.deb
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1704/x86_64/7fa2af80.pub
sudo apt-get update
sudo apt-get install cuda

# Post installation environment variables
echo "export PATH=/usr/local/cuda-9.0/bin\${PATH:+:\${PATH}}" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64 \${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}" >> ~/.bashrc

# Get CUDNN
sudo dpkg -i libcudnn7_7.0.3.11-1+cuda9.0_amd64.deb
sudo dpkg -i libcudnn7-dev_7.0.3.11-1+cuda9.0_amd64.deb
echo "export CUDA_HOME=/usr/lib/x86_64-linux-gnu" >> ~/.bashrc

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
sudo python -m pip install autograd
sudo pip install enum34

git clone --recursive git@github.com:roderickObrist/models.git

# modify as per diff patch
nano models/research/syntaxnet/dragnn/python/component.py
nano models/research/syntaxnet/tensorflow/tensorflow/core/framework/variant_op_registry.cc

cd models/research/syntaxnet/tensorflow
./configure
cd ..
bazel test ... --action_env=PYTHON_BIN_PATH=/usr/bin/python --action_env=PYTHON_LIB_PATH=/usr/local/lib/python2.7/dist-packages
