# FlagNet

This has been tested on **Ubuntu Server 14.04** running on a variety of EC2 instances. It hasn't been tested, but there's no theoretical reason it wouldn't work out-of-the-box on any other UNIX/Linux platform.

What follows are instructions to get a working model, starting from a blank EC2 instance (t2.micro in this case, but **a larger instance [or local machine] is highly recommended for compiling SyntaxNet**) running `ami-2d39803a`:

 1. Install the necessary packages:
 
  ```
  sudo add-apt-repository ppa:webupd8team/java
  sudo apt-get update
  sudo apt-get install git swig python-setuptools python-dev build-essential oracle-java8-installer
  sudo easy_install pip
  sudo pip install -U protobuf==3.0.0b2
  sudo pip install asciitree numpy
  ```
  
 2. Clone this repository and its submodules. This may take a while:

  ```
  git clone --recursive https://github.com/Undo1/FlagNet.git
  ```
  
 3. [Install Bazel](http://www.bazel.io/docs/install.html):
 
  ```
  echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
  curl https://storage.googleapis.com/bazel-apt/doc/apt-key.pub.gpg | sudo apt-key add -
  sudo apt-get update && sudo apt-get install bazel
  ```
  
 4. Configure and Install SyntaxNet:
 
  ```
  cd FlagNet/models/syntaxnet/tensorflow
  ./configure
  cd ..
  bazel test syntaxnet/... util/utf8/...
  ```
  
  This will also take a while. There might be precompiled versions, I'm not sure. 
