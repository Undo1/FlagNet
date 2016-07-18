# FlagNet

## Installation

This has been tested on **Ubuntu Server 14.04** running on a variety of EC2 instances. It hasn't been tested, but there's no theoretical reason it wouldn't work out-of-the-box on any other UNIX/Linux platform.

What follows are instructions to get a working model, starting from a blank EC2 instance (c4.2xlarge in this case, but a smaller instance should be fine, just slower) running `ami-2d39803a`:

 1. Install the necessary packages:
 
  ```
  sudo add-apt-repository ppa:webupd8team/java
  sudo apt-get update
  sudo apt-get install git swig ruby python-setuptools python-dev build-essential oracle-java8-installer
  sudo easy_install pip
  sudo pip install -U protobuf==3.0.0b2
  sudo pip install asciitree numpy flask
  ```
  
  Also [install TensorFlow](https://www.tensorflow.org/versions/r0.9/get_started/os_setup.html#pip-installation). For our case, we'll use the CPU-only version for 64-bit Linux:
  
  ```
  export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.9.0-cp27-none-linux_x86_64.whl
  sudo pip install --upgrade $TF_BINARY_URL
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
  
## Training

You should start out with a file that looks like this. More info on how to *get* to this is coming. There's a script you need that is written in Swift right now; I'm going to port it to something more common. Ping me (@Undo) in the usual places and I can send you the parsed file I have:

```
buy or
buy build
buy for deployment web
buy ?
```

<sup>(question title, not real flag data)</sup>

To train, `cd` into the `cnn-text-classification-tf` directory. Edit lines 34 and 36 in the `data_helpers.py` file to point to your parsed data files:

```
 33     # Load data from files
 34     positive_examples = list(open("/path/to/positive/examples.txt", "r").readlines())
 35     positive_examples = [s.strip() for s in positive_examples]
 36     negative_examples = list(open("/path/to/negative/examples.txt", "r").readlines())
```

In this case, the 'positive' examples would be helpful flags, while the 'negative' examples would be declined flags. After you've done this, run `python train.py`

Every 300 steps (configurable), the script runs an evaluation on data that wasn't included in the training set, and saves the current model as a 'checkpoint'. I see ~89% as a maximum accuracy in my tests after a few thousand steps.

When the accuracy of the evaluation reaches an acceptable mark, kill the script. Make a note of the model that was last saved (it'll look like `/home/ubuntu/FlagNet/cnn-text-classification-tf/runs/1468878881/checkpoints/model-900`).

## Running

Now that you have a trained file, edit `cnn-text-classification-tf/cnntest.py` to point to your checkpoints folder. In the example above, edit line 13 to be as follows:

```
tf.flags.DEFINE_string("checkpoint_dir", "/home/ubuntu/FlagNet/cnn-text-classification-tf/runs/1468878881/checkpoints", "Checkpoint directory from training run")
```

It will automatically choose the last saved checkpoint.

You're now ready to run the server. `cd` to `FlagNet/parsey-mcparseface-server` and run `python server.py`. By default, it will start on port 8000 (configurable in server.py). In your browser, go to `http://your.ip.goes.here:8000/?q=this%20is%20a%20test`.
