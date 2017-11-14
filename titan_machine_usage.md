# Using the Autolab Titan Machine

This guide has details on how to set up an account for the Autolab’s Titan X GPU cluster.
In addition, it includes important information on how to manage your Python packages and install new packages on this machine.
Please read this guide in full before logging onto the machine.

## Setting up a User Account

Email <mmatl@berkeley.edu> and <bderose@berkeley.edu> with the subject line “Triton Access” and the following three bits of information:
* Your full name.
* Your Berkeley email.
* Your desired username (this will be the name of your home directory).

We’ll get back to you once your account has been created. If you need sudo access, indicate that and describe why.

## Logging In

Once you receive an email confirming that your account has been created, simply log in via SSH like this:

```shell
ssh -AXY username@128.32.192.73 -p 8028
```

Accessing Triton II is just as easy:

```shell
ssh -AXY username@128.32.192.73 -p 8031
```

Your initial password will be ‘autolab’ -- change it as soon as you log in with the `passwd` command.

## Copying Files to the Machine

When copying files to the machine, remember to include the proper port number:

```shell
scp -P 8028 localfilename username@128.32.192.73:~/path/to/whatever
```

## Managing Python Installations

The number one rule of this machine is to NEVER run a sudo pip install:

```shell
sudo pip install blah # NEVER DO THIS!!!!!
```

Instead, always use a virtual environment to manage your Python packages and pip install anything you need into one of your virtualenvs.
The basics of virtualenvs are explained [here](http://python-guide-pt-br.readthedocs.io/en/latest/dev/virtualenvs/), but the basic ideas are as follows.
To create a virtual environment in a folder, run the following command:

```shell
virtualenv --system-site-packages env
```

This creates a virtual environment in the `./env` folder.
Make sure to include the `--system-site-packages` flag, as this will allow you to access some useful packages 
I’ve installed this at the global level for all users. If you know that you don’t want to use these packages, you can leave this flag off.
Whenever you want to run Python code, you need to activate the virtual environment:

```shell
source ./env/bin/activate
```

You should see a change in your bash prompt to indicate that you’re inside a virtual environment:

```shell
(env) $
```

Once you’ve activated your virtual environment, you’re free to install useful packages with pip:

```shell
pip install <packagename> # note -- no sudo!!!
```

If you would prefer to use conda or another virtual environment tool, feel free to use it -- just be careful to avoid installing anything in the global system’s Python site packages.
I want to keep that as clean as possible so that we don’t end up with versioning conflicts between different users.

## Pre-Installed Packages
While we want to keep most of your Python packages in your own virtual environments, there are a few things that everyone is likely to use and that benefit from being compiled from source.
I’ve installed a few of these packages in the global system’s Python site packages that you’re free to use – just include the `--system-site-packages` flag when you create your virtual environments to automatically include them.
Here’s a list of these packages – note that they’re subject to change in the future

* Tensorflow 1.4.0 – This is hand-compiled and optimized for this machine. I’d use it if you can – if you need an older version of Tensorflow, pip install it in your virtual environment.
  * Note -- this is installed for both Python 2.7 and Python 3.5.
* OpenCV 3.3 – This is hand-compiled and optimized for this machine. It also includes all of OpenCV's video processing capabilities. If you need a different version of OpenCV, just pip install it in your virtualenv.
  * Note -- this is currently only installed for Python 2.7. If you need it for 3.5, just pip install it.
* Ray 0.0.1 - An early version of a useful parallelization package from the RISE lab. Use as needed.

## Large Datasets
Since the machine only has two 1TB SSD’s, you should try to keep large datasets off of the machine except when you’re using them.
If you have a dataset larger than 100GB, please place it in `/mnt/data`, a 700GB partition exclusively allocated to data.

## Using Tensorflow
By default, Tensorflow expands to fill all available GPUs on our machine.
When you queue up a model to train, please limit each Tensorflow process to a single GPU by setting
the `CUDA_VISIBLE_DEVICES` environment variable to either 0, 1, 2, or 3 depending on which GPU is available.
To check which device is available, run the command `nvidia-smi`:

```shell
$ nvidia-smi
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 384.90                 Driver Version: 384.90                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  TITAN X (Pascal)    Off  | 00000000:01:00.0 Off |                  N/A |
| 23%   29C    P8     9W / 250W |      2MiB / 12189MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  TITAN Xp            Off  | 00000000:02:00.0 Off |                  N/A |
| 23%   36C    P8     9W / 250W |      2MiB / 12189MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   2  TITAN X (Pascal)    Off  | 00000000:03:00.0 Off |                  N/A |
| 23%   37C    P8     8W / 250W |      2MiB / 12189MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   3  TITAN X (Pascal)    Off  | 00000000:04:00.0  On |                  N/A |
| 23%   40C    P8    12W / 250W |    465MiB / 12186MiB |      1%      Default |
+-------------------------------+----------------------+----------------------+
```

The GPU entry in each output block indicates an ID number (0, 1, 2, or 3) and the Memory Usage block indicates how heavily the GPU is being used.
Choose the GPU that has the least usage for training your model. If they're all near capacity, wait for the other jobs to finish before starting up yours.

For example, given the above output, we would run
```shell
CUDA_VISIBLE_DEVICES=1 python train_my_model.py
```

Note that device 1 is a TITAN Xp, which is a bit faster than the Titan X (Pascal) GPUs. Use device 3 last, as it is running the displays and will therefore have slightly less memory.

If your job does not require an entire GPU, you can change the TensorFlow session config so that TF will only allocate memory as needed. This is very helpful as it allows running multiple TF sessions on the same GPU:
```python
tf_config = tf.ConfigProt() # fill in w/ custom config 
tf_config.gpu_options.allow_growth = True
sess = tf.Session(config=tf_config)
```
See more details [here](https://www.tensorflow.org/tutorials/using_gpu#allowing_gpu_memory_growth).

