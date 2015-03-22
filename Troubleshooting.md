### Pyrit does not use SSE2 ###

The CPU-core supports SSE2 since version '0.2.3'.

### '_ImportError: /usr/.../_cpyrit/_cpyrit\_cpu.so: cannot restore segment prot after reloc: Permission denied_' ###

This error may happen with SSE2-support compiled in and is caused by SELinux not trusting Pyrit's library. You can follow SELinux's guidelines on fixing that for your specific system. Or you can disable SELinux for the time being by executing (as root):

```
echo 0 > /selinux/enforce
```

The error was fixed in version '0.2.4 [r152](https://code.google.com/p/pyrit/source/detail?r=152)'.


### Using CPyrit-CUDA with CUDA 2.2 causes the error message '_Failed to load CUDA-core (CUDA\_ERROR\_INVALID\_IMAGE)_' ###

This error was fixed in version '0.2.3'. You either have to update to '0.2.3' or downgrade to CUDA 2.1.

### Using CPyrit-CUDA with CUDA 3.0 causes the error message '_SystemError: CUDA\_ERROR\_INVALID\_IMAGE_' ###

Please use the GPU-drivers that come with CUDA 3.0 at http://www.nvidia.com/object/cuda_get.html

### Pyrit no longer uses all CPUs after installing a GPU-driven extension-module ###

This behaviour is intended. Pyrit keeps one CPU free for scheduling work with every GPU it uses.

### GPU performance is reduced on system with Hyper Threading ###

NVidia/ATI GPU driver requires at least one real core per GPU for efficient work. With HT enabled, the driver needs to fight for CPU cycles with CPU computing core. This leads to GPU starvation and decreased performance. The problem can be solved by reducing number of running CPU-cores: Open '.pyrit/config' and set 'limit\_ncpus' to the number of physical CPU-cores.

### The GPU does not show up in '_list\_cores_' ###

Pyrit suppresses most errors that occur while loading the GPU-extensions as they are usually caused by simply not having compatible hardware installed. Open a terminal and try loading the offending module directly to get more information.

For CPyrit-OpenCL:

```
python -c 'from cpyrit import _cpyrit_opencl'
```

For CPyrit-CUDA:

```
python -c 'from cpyrit import _cpyrit_cuda'
```

There is no output at all if the module was loaded successfully. Some of the errors you might get include:

  * **`ImportError: cannot import name _cpyrit_cuda / ImportError: cannot import name _cpyrit_opencl`**

> The CPyrit-OpenCL or CPyrit-CUDA modules are simply not installed or can't be found in the current Python-environment. Try (re-) installing the extension modules and make sure you don't mix the Python-interpreters you use (this can happen on MacOS which tends to have multiple versions of Python installed).

  * **`ImportError: libcuda.so.1: cannot open shared object file: No such file or directory`**

> The CPyrit-CUDA modules requires Nvidia's proprietary driver to be installed. Depending on your distribution you may also need to symlink e.g. '/usr/lib/nvidia/libcuda.so.1' to '/usr/lib/libcuda.so.1' or add '/usr/lib/nvidia' to your _ldconfig_.

  * **`NVIDIA: could not open the device file /dev/nvidiactl (No such file or directory).`**

> Ensure that the 'nvidia' module has been successfully loaded. Taking a look at the kernel ringbuffer may give more help:

```
 modprobe nvidia
 dmesg | tail
```

  * **`ImportError: CUDA seems to be unavailable or no device reported.`**

> The CUDA-driver has loaded but failed to initialize or reports that none of your GPUs is compatible. Take a look at [this page](http://www.nvidia.com/object/cuda_learn_products.html) to find out if your GPU is supported. You may need to update your drivers.

### Xorg becomes slow and unresponsive when using Pyrit on the GPU ###

O rly ?

### Pyrit says it stored only Y passwords after trying to import a wordlist of X entries ###

Pyrit ignores passwords that have less than 8 or more than 63 characters as those can't be used for WPA anyway.

### The .pyrit directory has a size of only Y MBs after importing a wordlist of X GBs ###

Pyrit uses [zlib](http://www.zlib.net)-compression to store the passwords.

### There is another problem with Pyrit ###

Create a ticket in the [Issue-Tracker](http://code.google.com/p/pyrit/issues/list) or write an e-mail to lukas.lueg@gmail.com