# Installation Instructions

*Start with cuda*
http://docs.nvidia.com/cuda/cuda-installation-guide-linux/#choose-installation-method

*Download the .dpkg*
https://developer.nvidia.com/cuda-downloads

```
sudo apt-get install linux-headers-$(uname -r)
sudo dpkg -i cuda-repo-ubuntu1704_9.0.176-1_amd64.deb
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1704/x86_64/7fa2af80.pub
sudo apt-get update
sudo apt-get install cuda
echo "export PATH=/usr/local/cuda-9.0/bin\${PATH:+:\${PATH}}" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64 \${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}" >> ~/.bashrc
```
