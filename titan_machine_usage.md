# Using the Autolab Titan Machine

This guide has details on how to set up an account for the Autolab’s Titan X GPU cluster.
In addition, it includes important information on how to manage your Python packages and install new packages on this machine.
Please read this guide in full before logging onto the machine.

## Setting up a User Account

Email <mmatl@berkeley.edu> with the subject line “Triton Access” and the following three bits of information:
* Your full name.
* Your Berkeley email.
* Your desired username (this will be the name of your home directory).

I’ll get back to you once your account has been created. If you need sudo access, indicate that and tell me why.

## Logging In

Once you receive an email confirming that your account has been created, simply log in via SSH like this:

```shell
ssh -AXY username@128.32.192.73 -p 8028
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

* Tensorflow 1.1.0rc2 – This is hand-compiled and optimized for this machine. I’d use it if you can – if you need an older version of Tensorflow, pip install it in your virtual environment.
* OpenCV 3.3 – This is hand-compiled and optimized for this machine. It also includes all of OpenCV's video processing capabilities. If you need a different version of OpenCV, just pip install it in your virtualenv.
* Ray 0.0.1 - An early version of a useful parallelization package from the RISE lab. Use as needed.

## Large Datasets
Since the machine only has two 1TB SSD’s, you should try to keep large datasets off of the machine except when you’re using them.
If you have a dataset larger than 100GB, please place it in `/mnt/data`, a 700GB partition exclusively allocated to data.


