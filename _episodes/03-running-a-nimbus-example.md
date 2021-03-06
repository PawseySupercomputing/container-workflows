---
title: "Running a Container App in Nimbus"
teaching: 10
exercises: 35
questions:
- "How do you use Containers on Nimbus?"
objectives:
- "Setup and install Docker"
- "Use a Docker container to run a Machine Learning script in Tensorflow"
keypoints:
- "We launched a VM and ran a container to run a script."
- "We used Tensorflow without any Tensorflow installation work! Neat!"
---

## Let's get started with Nimbus

Before we begin the container training today we will need to setup a Nimbus Virtual Machine.  You can follow the Nimbus training at:

### [https://pawseysupercomputing.github.io/using-nimbus/](https://pawseysupercomputing.github.io/using-nimbus/) ###

Completing sections 1 to 6 will give you a Virtual Machine suitable for this lesson.  When you VM is up and running, come back to this page. Good luck!

## First we need to setup and install Docker

So now you have a working VM (or maybe you already had one good to go).  In section 2 we discussed the various tools for running containers.  You can use any tool you like on Nimbus and in this section we will use Docker.  First we need to setup a local repository for Docker, we can then install Docker, and then we can use a Docker container. These instructions are taken from https://docs.docker.com/install/linux/docker-ce/ubuntu/ and are replicated here with example output


## Set up the Repository

We will use the Apt tool in Ubuntu to install Docker, so first we need to make sure that the Apt package database is up to date.  You would normally do this when coming to your VM after a while, or if you made some other change that might require updates.  It's always safe to run this command, so do so frequently.

~~~
$ sudo apt-get update
~~~
{: .source}

You will see an response that looks like this the following (we've removed a few lines for brevity).

~~~
Get:1 http://security.ubuntu.com/ubuntu bionic-security InRelease [83.2 kB]
Get:2 http://nova.clouds.archive.ubuntu.com/ubuntu bionic InRelease [242 kB]                     
Get:3 http://security.ubuntu.com/ubuntu bionic-security/main Sources [19.6 kB]
...etc...
Get:31 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 Packages [1728 B]
Get:32 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/multiverse Translation-en [1236 B]
Fetched 26.3 MB in 15s (1713 kB/s)                                                               
Reading package lists... Done
~~~
{: .output}


With that done we now need to install certificates for getting the Docker repository.  First we will install the software dependancies:

~~~
$ sudo apt-get install apt-transport-https  ca-certificates  curl  software-properties-common
~~~
{: .source}

When apt-get asks "Do you want to continue? [Y/n]" you will need to enter Y and hit return.

~~~
Reading package lists... Done
Building dependency tree       
Reading state information... Done
ca-certificates is already the newest version (20180409).
The following package was automatically installed and is no longer required:
  grub-pc-bin
Use 'sudo apt autoremove' to remove it.
The following additional packages will be installed:
  libcurl4 python3-software-properties
The following NEW packages will be installed:
  apt-transport-https
The following packages will be upgraded:
  curl libcurl4 python3-software-properties software-properties-common
4 upgraded, 1 newly installed, 0 to remove and 23 not upgraded.
Need to get 407 kB of archives.
After this operation, 152 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://nova.clouds.archive.ubuntu.com/ubuntu bionic/universe amd64 apt-transport-https all 1.6.1 [1692 B]
Get:2 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/main amd64 curl amd64 7.58.0-2ubuntu3.1 [159 kB]
Get:3 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libcurl4 amd64 7.58.0-2ubuntu3.1 [214 kB]
Get:4 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/main amd64 software-properties-common all 0.96.24.32.2 [9916 B]
Get:5 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/main amd64 python3-software-properties all 0.96.24.32.2 [22.8 kB]
Fetched 407 kB in 3s (134 kB/s)                   
Selecting previously unselected package apt-transport-https.
(Reading database ... 59930 files and directories currently installed.)
Preparing to unpack .../apt-transport-https_1.6.1_all.deb ...
Unpacking apt-transport-https (1.6.1) ...
Preparing to unpack .../curl_7.58.0-2ubuntu3.1_amd64.deb ...
Unpacking curl (7.58.0-2ubuntu3.1) over (7.58.0-2ubuntu3) ...
Preparing to unpack .../libcurl4_7.58.0-2ubuntu3.1_amd64.deb ...
Unpacking libcurl4:amd64 (7.58.0-2ubuntu3.1) over (7.58.0-2ubuntu3) ...
Preparing to unpack .../software-properties-common_0.96.24.32.2_all.deb ...
Unpacking software-properties-common (0.96.24.32.2) over (0.96.24.32.1) ...
Preparing to unpack .../python3-software-properties_0.96.24.32.2_all.deb ...
Unpacking python3-software-properties (0.96.24.32.2) over (0.96.24.32.1) ...
Setting up apt-transport-https (1.6.1) ...
Setting up libcurl4:amd64 (7.58.0-2ubuntu3.1) ...
Processing triggers for libc-bin (2.27-3ubuntu1) ...
Processing triggers for man-db (2.8.3-2) ...
Setting up python3-software-properties (0.96.24.32.2) ...
Processing triggers for dbus (1.12.2-1ubuntu1) ...
Setting up software-properties-common (0.96.24.32.2) ...
Setting up curl (7.58.0-2ubuntu3.1) ...
~~~
{: .output}

With this successfully completed we can get the GPG key from Docker:
~~~
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
~~~
{: .source}

If this works, it won't be very chatty:
~~~
OK
~~~
{: .output}


We need to verify that this is all fine, as follows using the apt-key fingerprint search tool:
~~~
$ sudo apt-key fingerprint 0EBFCD88
~~~
{: .source}

~~~
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
~~~
{: .output}

Let's check something here before we continue.  We need to make sure we're using the right version Ubuntu:
~~~
$ lsb_release -cs
~~~
{: .source}

If you've run your VM from an Ubuntu 16.04 LTS version you will see this:
~~~
xenial
~~~
{: .output}

If you don't see xenial, grab an instructor to assist, or consult with Google on what the problem might be.  That command (lsb_release) shows up in the next command, so if something fails check back here.

Now we can add the local repository for Docker, we need to do this because apt-get doesn't yet know how to find Docker CE.  Just copy the following command with all the punctuation (it's all important).
~~~
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
~~~
{: .source}

~~~
Get:1 http://security.ubuntu.com/ubuntu bionic-security InRelease [83.2 kB]                                                
Get:2 https://download.docker.com/linux/ubuntu bionic InRelease [64.4 kB]                                                  
Hit:3 http://nova.clouds.archive.ubuntu.com/ubuntu bionic InRelease                                             
Get:4 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates InRelease [83.2 kB]
Hit:5 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-backports InRelease
Get:6 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [100 kB]
Get:7 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [62.1 kB]
Fetched 393 kB in 4s (91.2 kB/s)  
Reading package lists... Done
~~~
{: .output}

Great! So far so good, next step: installing Docker.

## Now we can install Docker (Community Edition)

We're going to install Docker CE (Community Edition).  There is also an Enterprise Edition ('Enterprise Edition' is code for 'costs you money', 'Community Edition' is code for free, and you're on your own if something goes wrong).  today we'll be installing the Community Edition, this will usually be the edition you work with in research.

Since we added the repository above we need to re-run apt-get update so that apt can find the Docker installation materials:
~~~
$ sudo apt-get update
~~~
{: .source}

~~~
Get:1 http://security.ubuntu.com/ubuntu bionic-security InRelease [83.2 kB]                  
Hit:2 https://download.docker.com/linux/ubuntu bionic InRelease                                          
Hit:3 http://nova.clouds.archive.ubuntu.com/ubuntu bionic InRelease                                                           
Hit:4 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-updates InRelease
Hit:5 http://nova.clouds.archive.ubuntu.com/ubuntu bionic-backports InRelease
Fetched 83.2 kB in 2s (39.8 kB/s)
Reading package lists... Done
~~~
{: .output}

And now we can install Docker.
~~~
$ sudo apt-get install docker-ce
~~~
{: .source}

~~~
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  aufs-tools cgroupfs-mount libltdl7 pigz
Suggested packages:
  mountall
The following NEW packages will be installed:
  aufs-tools cgroupfs-mount docker-ce libltdl7 pigz
0 upgraded, 5 newly installed, 0 to remove and 24 not upgraded.
Need to get 34.2 MB of archives.
After this operation, 182 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://nova.clouds.archive.ubuntu.com/ubuntu xenial/universe amd64 pigz amd64 2.3.1-2 [61.1 kB]
Get:2 https://download.docker.com/linux/ubuntu xenial/stable amd64 docker-ce amd64 18.03.1~ce-0~ubuntu [34.0 MB]
Get:3 http://nova.clouds.archive.ubuntu.com/ubuntu xenial/universe amd64 aufs-tools amd64 1:3.2+20130722-1.1ubuntu1 [92.9 kB]
Get:4 http://nova.clouds.archive.ubuntu.com/ubuntu xenial/universe amd64 cgroupfs-mount all 1.2 [4,970 B]
Get:5 http://nova.clouds.archive.ubuntu.com/ubuntu xenial/main amd64 libltdl7 amd64 2.4.6-0.1 [38.3 kB]
Fetched 34.2 MB in 3s (10.7 MB/s)  
Selecting previously unselected package pigz.
(Reading database ... 54012 files and directories currently installed.)
Preparing to unpack .../pigz_2.3.1-2_amd64.deb ...
Unpacking pigz (2.3.1-2) ...
Selecting previously unselected package aufs-tools.
Preparing to unpack .../aufs-tools_1%3a3.2+20130722-1.1ubuntu1_amd64.deb ...
Unpacking aufs-tools (1:3.2+20130722-1.1ubuntu1) ...
Selecting previously unselected package cgroupfs-mount.
Preparing to unpack .../cgroupfs-mount_1.2_all.deb ...
Unpacking cgroupfs-mount (1.2) ...
Selecting previously unselected package libltdl7:amd64.
Preparing to unpack .../libltdl7_2.4.6-0.1_amd64.deb ...
Unpacking libltdl7:amd64 (2.4.6-0.1) ...
Selecting previously unselected package docker-ce.
Preparing to unpack .../docker-ce_18.03.1~ce-0~ubuntu_amd64.deb ...
Unpacking docker-ce (18.03.1~ce-0~ubuntu) ...
Processing triggers for man-db (2.7.5-1) ...
Processing triggers for libc-bin (2.23-0ubuntu10) ...
Processing triggers for ureadahead (0.100.0-19) ...
Processing triggers for systemd (229-4ubuntu21.2) ...
Setting up pigz (2.3.1-2) ...
Setting up aufs-tools (1:3.2+20130722-1.1ubuntu1) ...
Setting up cgroupfs-mount (1.2) ...
Setting up libltdl7:amd64 (2.4.6-0.1) ...
Setting up docker-ce (18.03.1~ce-0~ubuntu) ...
Processing triggers for libc-bin (2.23-0ubuntu10) ...
Processing triggers for systemd (229-4ubuntu21.2) ...
Processing triggers for ureadahead (0.100.0-19) ...
~~~
{: .output}

If you see errors or other failures, double check that the Docker repository was installed properly and work your way back through the previous commands; the responses are quite verbose, so it's easy to miss something unexpected happening.

Let's check that everything installed properly with a Hello world test (literally called hello-world).
~~~
$ sudo docker run hello-world
~~~
{: .source}

~~~
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9bb5a5d4561a: Pull complete
Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
~~~
{: .output}

If you get the 'Hello from Docker' message then all is well.


## Running a container for machine learning

Now we're going to run a container to perform a machine learning benchmark application.  This will be an arbitrary example for most people.  Don't worry, we're not going to get into the details of the machine learning (by all means feel free to explore), but we are going to use this as an example of running a command with a container.  Hopefully you will see the relevance to your own workflows.

First let's get a sample program and the data that it needs:
~~~
git clone https://github.com/tensorflow/models.git
~~~
{: .source}

This will download some data and code:
~~~
Cloning into 'models'...
remote: Counting objects: 17327, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 17327 (delta 2), reused 2 (delta 2), pack-reused 17323
Receiving objects: 100% (17327/17327), 469.82 MiB | 5.41 MiB/s, done.
Resolving deltas: 100% (10311/10311), done.
Checking connectivity... done.
Checking out files: 100% (2248/2248), done.
~~~
{: .output}

If this downloaded correctly you can have a look at the script we will use (called **convolutional.py**).  Do this and make sure that it's there:
~~~
$ ls models/tutorials/image/mnist
~~~
{: .source}

~~~
BUILD  convolutional.py  __init__.py
~~~
{: .output}

If you can see it, then we have some minor editing to complete. We need to edit **convolutional.py** to set NUM_EPOCS to 1 (since we don’t have a GPU here, this will take forever at the current value of 10).  If you need assistance with this in a class, call a class assistance, otherwise have a look at (link).

Find the line with the variable *NUM_EPOCHS**, you should see that it's set to 10. Make the change to 1 and save the file. It now will look like the following:

~~~
# CVDF mirror of http://yann.lecun.com/exdb/mnist/
SOURCE_URL = 'https://storage.googleapis.com/cvdf-datasets/mnist/'
WORK_DIRECTORY = 'data'
IMAGE_SIZE = 28
NUM_CHANNELS = 1
PIXEL_DEPTH = 255
NUM_LABELS = 10
VALIDATION_SIZE = 5000  # Size of the validation set.
SEED = 66478  # Set to None for random seed.
BATCH_SIZE = 64
NUM_EPOCHS = 1
EVAL_BATCH_SIZE = 64
EVAL_FREQUENCY = 100  # Number of steps between evaluations.
~~~
{: .output}

Now we can get Docker involved, and there's only one command to start Docker, get the container (Tensorflow) and have it see our data directory and it's this:
~~~
$ sudo docker run --rm -it -v /home/ubuntu/models/tutorials/image/mnist/:/notebooks tensorflow/tensorflow bash
~~~
{: .source}

What's happening here?
1.  We're running docker (**docker run**),
2.  We're telling docker to mount our MINST code+data on the /notebooks directory in the container (**-v /home/ubuntu/models/tutorials/image/mnist/:/notebooks**),
3.  We're telling docker to get the TensorFlow container from a user called tensorflow (**tensorflow/tensorflow**),
4.  Finally we're telling the notebook to start a bash session so we can give it commands (**bash**).

There will be output from Docker as it finds the Tensorflow container and gets everything ready.  If everything proceeds according to plan you will end up with something like this:
~~~
root@503d2d42cc5c:/notebooks#
~~~
{: .output}

All going well, you've just installed Tensorflow, a machine learning framework (one of several commonly used).  Take a moment to think about what you didn't do.
1. You didn't do searching and reading on the installation requirements for Tensorflow (**time saved**),
2. You didn't think about whether your VM had the dependancies all set up (**time saved**),
3. You didn't use apt-get or anything like that to install anything on your VM (**time and sanity saved**).

Let's check that it really did what we wanted. If we look at the local **notebooks/** directory now we will the contents we saw before:

~~~
root@503d2d42cc5c:/notebooks# ls
~~~
{: .source}

~~~
BUILD  __init__.py  convolutional.py
~~~
{: .output}

## Let's machine learn all the things!

Now we can simply run our example (it's called **convolutional.py** and it runs in python).  This will be running in our Tensorflow container. We don't have to do anything special for the command to use tensorflow, in this case the script is expecting to find the right environment.  It should take about 5 mins, so take a few moments to breathe.
~~~
root@503d2d42cc5c:/notebooks# python convolutional.py
~~~
{: .source}

~~~
/usr/local/lib/python2.7/dist-packages/h5py/__init__.py:36: FutureWarning: Conversion of the second argument of issubdtype from `float` to `np.floating` is deprecated. In future, it will be treated as `np.float64 == np.dtype(float).type`.
  from ._conv import register_converters as _register_converters
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting data/train-images-idx3-ubyte.gz
Extracting data/train-labels-idx1-ubyte.gz
Extracting data/t10k-images-idx3-ubyte.gz
Extracting data/t10k-labels-idx1-ubyte.gz
2018-05-26 08:59:55.416198: I tensorflow/core/platform/cpu_feature_guard.cc:140] Your CPU supports instructions that this TensorFlow binary was not compiled to use: FMA
Initialized!
Step 0 (epoch 0.00), 8.6 ms
Minibatch loss: 8.334, learning rate: 0.010000
Minibatch error: 85.9%
Validation error: 84.6%
Step 100 (epoch 0.12), 449.7 ms
Minibatch loss: 3.240, learning rate: 0.010000
Minibatch error: 4.7%
Validation error: 7.6%
Step 200 (epoch 0.23), 443.6 ms
Minibatch loss: 3.371, learning rate: 0.010000
Minibatch error: 10.9%
Validation error: 4.7%
Step 300 (epoch 0.35), 443.7 ms
Minibatch loss: 3.170, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 3.2%
Step 400 (epoch 0.47), 444.7 ms
Minibatch loss: 3.222, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 2.6%
Step 500 (epoch 0.58), 443.5 ms
Minibatch loss: 3.173, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 2.7%
Step 600 (epoch 0.70), 452.4 ms
Minibatch loss: 3.125, learning rate: 0.010000
Minibatch error: 4.7%
Validation error: 2.1%
Step 700 (epoch 0.81), 444.8 ms
Minibatch loss: 3.006, learning rate: 0.010000
Minibatch error: 3.1%
Validation error: 2.2%
Step 800 (epoch 0.93), 446.5 ms
Minibatch loss: 3.065, learning rate: 0.010000
Minibatch error: 6.2%
Validation error: 1.9%
Test error: 1.8%
~~~
{: .output}

## Good work!
So we installed Docker and grabbed a container from the internet and run a command using the container!

Now we can get on to the next step:  using this container to run the same workflow on a HPC facility.
