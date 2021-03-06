---
layout: post
title:  nVidia CUDA multi-core development in Ubuntu
tags:   how-to linux programming

summary: >
    Here's a step-by-step guide on how to install nVidia's CUDA SDK
    for multi-core development on Ubuntu 9.10 (and possibly later
    versions).

cover_img_filename:
    https://lh6.googleusercontent.com/-aO3oKZEMCpg/TM3JTfO7giI/AAAAAAAACWg/H6o3MoxiAYY/s144/Screenshot.png
---

Here's a step-by-step guide on how to install nVidia's CUDA SDK for multi-core
development on Ubuntu 9.10 (and possibly later versions).

# Download the driver, toolkit, and SDK

1. Go to the [CUDA download page][cudadownload] and click the *Get latest CUDA
   toolkit* link
2. Under the *Linux* section, find the latest *Ubuntu* packages
3. Download the latest Ubuntu package for your architecture, and remember where
   you put it

[cudadownload]:http://developer.nvidia.com/cuda-downloads

# Install the CUDA-enabled nVidia driver

1. On your GNOME panel, go to *System -> Administration -> Hardware Drivers*
2. Disable your current nVidia driver
3. Enter a virtual terminal by pressing *Ctrl + Alt + F1*
4. Log in to your virtual terminal
5. Kill X and GDM

{% highlight sh %}
sudo service gdm stop
{% endhighlight %}

6. `cd` to where you downloaded the driver
7. Run the driver installer script

{% highlight sh %}
sudo ./nvidia-driver-install-script-name.run
{% endhighlight %}

8. Follow the on-screen instructions to install the driver. Allow it to
   automatically configure your `xorg.config` for you.
9. After the installer has finished, reboot

{% highlight sh %}
sudo reboot
{% endhighlight %}

# Install the CUDA toolkit

1. Open a terminal and `cd` to where you donloaded the toolkit
2. Run the toolkit installer

{% highlight sh %}
sudo ./nvidia-toolkit-install-script-name.run
{% endhighlight %}

# Add CUDA to your environment variables

1. Open a terminal and type

{% highlight sh %}
echo "# CUDA stuff
PATH=\$PATH:/usr/local/cuda/bin
LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/usr/local/cuda/lib
export PATH
export LD_LIBRARY_PATH" >> ~/.bashrc
{% endhighlight %}

2. Reload your `.bashrc` by either restarting your terminal, or with

{% highlight sh %}
. ~/.bashrc
{% endhighlight %}

# Install the CUDA SDK

1. Open a terminal and `cd` to where you downloaded the SDK
2. Run the toolkit installer

{% highlight sh %}
sudo ./nvidia-SDK-install-script-name.run
{% endhighlight %}

# Install and configure GCC v4.3

At the time that this was written, GCC v4.4 had issues compiling CUDA code. You
must first install and configure GCC v4.3. (Don't worry, you can still keep
v4.4 and change it back whenever you want.)

1. Open a terminal and install gcc v4.3 from APT

{% highlight sh %}
sudo apt-get install gcc-4.3
{% endhighlight %}

2. `cd` to `/usr/bin`
3. Remove the current gcc and g++ symlinks

{% highlight sh %}
sudo rm gcc g++
{% endhighlight %}

4. Create new symlinks to v4.3

{% highlight sh %}
ln -s gcc-4.3 gcc
ln -s g++-4.3 g++
{% endhighlight %}

# Disable Compiz

Many of the CUDA examples are OpenGL-based, and Compiz has a tendency to
interfere.

1. Click *System -> Preferences -> Appearance*
2. Disable Compiz effects by clicking on the *Visual Effects* tab and selecting
   *none*

# Build and run CUDA examples
1. `cd` to the *C* directory in the CUDA SDK installation directory

{% highlight sh %}
cd ~/NVIDIA_GPU_Computing_SDK/C
{% endhighlight %}

2. Build the examples. This will take a few minutes to compile.

{% highlight sh %}
sudo make
{% endhighlight %}

3. Run the examples in `~/NVIDIA_GPU_Computing_SDK/C/bin/linux/release
4. Have fun!
