---
layout: post
title: Setting up an NVIDIA GPU card on Linux
tags: GPGPU linux
---

This post documents how I set up an NVIDIA CUDA GPU card on linux, specifically for CUDA computing (i.e. using it solely for GPGPU purposes). I already had a separate (AMD) graphics card I used for video output, and I wanted the NVIDIA card to be used only for computation, with no video use.

I found the whole process of setting up this card under linux to be problematic. NVIDIA's own guidance on their website did not seem to work, and in the end it required patching together bits of information from different sources. Here it is for future record:

Card: NVIDIA Quadro K1200 (PNY Low profile version)
Computer: HP desktop, integrated graphics on motherboard (Disabled in BIOS - though in hindsight I don't know if this was really necessary.)
Linux versions: Attempted on Fedora 23 (FAIL), Scientific Linux 6.7 (OK), CentOS 7.2 (OK)

## 1st Attempt: Using the NVIDA rpm package (FAIL)

This is the recommended installation route from NVIDIA. Bascially you download the relevant package manager install package. I was using CentOS so downloaded the RHEL/CentOS 7 `.rpm` file. You then add this to your package manager (e.g. yum). For RHEL/CentOS, you must have the `epel-release` repository enabled in yum:

`yum install epel-release`

Then you add the rpm package downloaded from nvidia:

`rpm --install cuda-repo<...>.rpm`

Followed by:

{% highlight bash %}
yum clean expire-cache
yum install cuda
{% endhighlight %}

It will install a load of package dependencies, the CUDA package, as well as the proprietary NVIDIA drivers for the card. I rebooted, only to find I could no longer launch CentOS in graphical mode. It would hang when trying to load the X-server driver files on boot. Only a text interface login was possible. Further playing around with the linux system logs showed there was a conflict with some of the OpenGL X11 libraries being loaded. 

I reverted to the earlier working state by launching in text mode and using `yum history undo` to revert all the installed packages in the previous step.

## 2nd Attempt: Using the NVIDIA runfile shell script

A second alternative is provided by NVIDIA, involving a shell script that installs the complete package as a platform-independent version. It bypasses the package manager completely and installs the relevant headers and drivers "manually". NVIDIA don't recommend this unless they don't supply a ready-made package for your OS, but I had already tried packages for Scientific Linux/RedHat, CentOS, and Fedora without success.

Before you go anywhere near the NVIDIA runfile.sh script, you have to blacklist the `nouveau` drivers that will may be installed. These are open source drivers for NVIDIA cards, but will create conflicts if you try to use them alongside the proprietary NVIDIA ones.

You blacklist them by adding a blacklist file to the modprobe folder, which controls which drivers load at the linux boot-up.

`vim /etc/modprobe.d/blacklist-nouveau.conf`

Add the following lines:

{% highlight bash %}
blacklist nouveau
options modeset=0
{% endhighlight %}

Now rebuild the startup script with:

`dracut --force`

Now the computer has to be restarted in text mode. The install script cannot be run if the desktop or X server is running. To do this I temporarily disabled the graphical/desktop service from starting up using `systemctl`, like this:

`systemctl set-default multi-user.target`

Then reboot. You'll be presented with a text-only login interface. First check that the nouveau drivers haven't been loaded:

`lsmod | grep nouveau` Should return a blank. If you get any reference to nouveau in the output, something has gone wrong when you tried to blacklist the drivers. Onwards...

Navigate to your NVIDIA runfile script after logging in. Stop there. 

Buried in the NVIDIA documentation is an important bit of information if you are planning on running the GPU for CUDA processing only, i.e. a separate, standalone card for GPGPU use, with another card for your video output. Theu note that installing the OpenGL library files can cause conflicts with the X-window server (Now they tell us!), but an option flag will disable their installation. Run the install script like so:

`sh cuda_<VERSION>.run --no-opengl-libs`

The option at the end is critical for it to work. I missed it off during one previous failed attempt and couldn't properly uninstall what I had done. The runfile does have an `--uninstall` option, but it's not guaranteed to undo everything.

You'll be presented with a series of text prompts, read them, but I ended up selecting 'yes' to most questions, and accepting the default paths. Obviously you should make ammendments for your own system. I would recommend installing the sample programs when it asks you so you can check the installation has worked and the card works as expected.

After that has all finished, you need to set some environment paths in your .bash_profile file. Add the following:

{% highlight bash %}
PATH=$PATH:/usr/local/cuda-7.5/bin
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-7.5/lib64
{% endhighlight %}

If you have changed the default paths during the installation process, ammend the above lines to the paths you entered in place of the defaults.

Now, you have to remember to restore the graphical/desktop service during boot up. (Assuming you used the systemctl method above). Restore with:

`systemctl set-default graphical.target`

Then reboot. It should work, hopefully!

Assuming you can login into your desktop without problems, you can double check the card is running fine, and can execute CUDA applications by compiling one of the handy sample CUDA applications called `deviceQuery`. Navigate to the path where you installed the sample CUDA programs, go into the utilities folder, into deviceQuery, and run `make`. You will get an application called deviceQuery that prints out lots of information about your CUDA graphics card. There are loads of other sample applications (less trivial than this one) that you can also compile and test in these folders. 

Remember, if you have followed the above steps, you can only use your CUDA card for computation, not graphical output. 


