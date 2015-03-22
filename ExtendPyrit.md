### Introduction ###

Adding support for more hardware-platforms to Pyrit is **very** easy. There are only a few steps besides the obvious requirement to write a library that implements the [PBKDF2](http://en.wikipedia.org/wiki/PBKDF2)-[HMAC](http://en.wikipedia.org/wiki/Hmac)-[SHA-1](http://en.wikipedia.org/wiki/SHA_hash_functions)-algorithm used for computing Pairwise Master Keys. This boils down to having the fastest (or most parallel) possible implementation of [SHA-1](http://en.wikipedia.org/wiki/SHA_hash_functions).

Pyrit abstracts access to the hardware in roughly three steps:
  * Direct access to the hardware is provided through Python-extensions usually written in C. These extension-modules encapsulate the hardware-platform in very minimal Python-classes which in turn provide a single function named `solve`. This function takes strings (passwords) and computes the corresponding Pairwise Master Keys.
  * The extension-modules should hide their implementation-details by getting sub-classed from the class `Core` provided in the `cpyrit`-module. The purpose of `Core` is to attach to Pyrit's scheduling routine, run self-tests, provide statistics and such. We should never need to know what exact kind of hardware we are actually talking to when using an instance of `Core`.
  * Right now we only have a bunch of classes that can compute Pairwise Master Keys for us. The glue that holds everything together is implemented as the almighty `CPyrit`-class which is the veil between hardware and client. All you need to worry about as a hardware-provider is how to tell `CPyrit` about your new module. All you got to do as a client is how to put work on the queue and get the results back. The magic in between is done by `CPyrit`.


### Talking to hardware ###

All extension-modules that provide hardware-access usually reside as part of the package `cpyrit`. The extension-modules should be very convenient about errors and take great care not to disrupt Pyrit in an unexpected way or method that is not common to all other modules. It must be possible to have it installed on a platform that does not support the hardware the module was written for. For example it must be possible to have static bindings to other libraries which may not be present on the platform Pyrit is executed on. As a general rule of thumb the modules should cause an `ImportError` in it's `init`-function if it fails (or does not want) to load for reasonable causes. In such case the `CPyrit`-class described further below swallows the exception and continues to initialize the other modules. If the module fails in an unexpected way, it may throw a `SystemError`-Exception which walks all the way up to Python`s exception-handler (and usually causes Pyrit to crash and burn as it should in such cases).

Some points to consider when writing a module for new hardware:
  * The module should provide a class that encapsulates access to the underlying hardware. If the hardware-platform itself provides multiple independent cores (e.g. possibly two or more GPUs), the class should be designed to get instantiated exactly once for every hardware-core.
  * The module should provide a function to enumerate available hardware-cores if applicable.
  * The core-functionality of computing Pairwise Master Keys should be implemented as function that is part of the class (not the module itself).
  * For Pyrit`s current implementation, the class does not need to be thread-safe. Unnecessary global variables should be avoided though.
  * The `solve`-function must take an ESSID and any [sequence](http://docs.python.org/c-api/sequence.html) (or even [iterable](http://docs.python.org/c-api/iter.html)) as parameters and return a [sequence](http://docs.python.org/c-api/sequence.html) (e.g. a tuple) of Pairwise Master Keys as strings of 32 bytes each.
  * The `solve`-function should release the [Global Interpreter Lock](http://effbot.org/pyfaq/what-is-the-global-interpreter-lock.htm) for most of it's runtime for obvious reasons of parallelism; this usually requires to copy the parameters to intermediate buffers. Don`t make any assumptions about objects after releasing the GIL and be sure not to talk to the Python-API at all without it.

This document will not go any deeper into how to write extension modules for Python. There is some [really great documentation](http://docs.python.org/extending/) about CPython's API on python.org. Pyrit's subversion-repository also includes an minimal 'hardware'-module named `cpyrit_null` that can be used as a guideline for those who are unfamiliar with writing extension-modules for Python.


### The Core-class ###

Every hardware-module may introduce it's own kind of limits and constraints due to details of the implementation or restrictions in the underlying hardware-platform. The `Core`-class hides all this in order to make the hardware-modules available to Pyrit's scheduling-routine more easily.

First of all, the `Core`-class is a sub-class of Python's `threading.Thread` so every instance of every sub-class of `Core` lives in it's own thread. The instances usually spend most of their time in `Thread`'s run()-function, trying to gather work (passwords) from the global work-queue, computing the corresponding results (Pairwise Master Keys) and pushing those back to the queue. The `Core`-class already provides this functionality. It also tries to calibrate itself so every call to `solve` takes exactly three seconds of wall-clock-time. This usually leads to good efficiency on the hardware-side (small overhead per call to hardware) and reasonable interactivity.

All that sub-classes of `Core` must do is to set the `.name`-attribute to a human-readable description of the underlying hardware-platform. They may also need to set the `.minBufferSize`- and `.maxBufferSize`-attributes to values arbitrary to the underlying hardware-platform. For example the `StreamCore`-class sets `.maxBufferSize` to 8192 because the current implementation for ATI-Stream can only take exactly that amount of passwords per call to hardware.

The following examples shows how the `Core`-class for Nvidia-CUDA is defined:
```
class CUDACore(Core, _cpyrit_cuda.CUDADevice):
    """Computes results on Nvidia-CUDA capable devices."""
    def __init__(self, queue, dev_idx):
        Core.__init__(self, queue)
        _cpyrit_cuda.CUDADevice.__init__(self, dev_idx)
        self.name = "CUDA-Device #%i '%s'" % (dev_idx+1, self.deviceName)
        self.minBufferSize = 1024
        self.buffersize = 4096
        self.maxBufferSize = 40960
        self.start()
```
Things to note here:
  * The new class `CUDACore` is a sub-class of `Core` (providing default-`run()`) and `_cpyrit_cuda.CUDADevice` (providing `solve`).
  * The argument `queue` (an instance of `CPyrit`) is passed to the `__init__`-function of `Core`, the argument `dev_idx` is passed to `CUDADevice`. Valid values of `dev_idx` are enumerated through `_cpyrit_cuda.listDevices()` later on in `CPyrit`(see below).
  * The call to `start()` enters the scheduling-routine provided by `run()` after setting the buffer-sizes to reasonable values for CUDA-platforms.
  * The instance daemonizes itself. It never exits but usually gets killed by the interpreter while waiting for more work.

### Everthing put together: The almighty CPyrit ###

Instances of `CPyrit` enumerate the available hardware-modules, instantiate them if possible and provide scheduling between the hardware and the caller. Although neither side should ever need to care about the inner workings of `CPyrit`, you should take note of some design goals of it's current implementation:
  1. We assume that there is an endless amount of work waiting to be put on the queue.
  1. We assume that there is no further (bandwidth-) latency inside `CPyrit`.
  1. We assume that instances of `Core` have different speeds, must be able to return results in random order and must be able to get more work any time.
  1. Callers of `CPyrit` can enqueue passwords by calling the `.enqueue()`-function. The function usually does not block and can be called many times before ever calling `.dequeue()`.
  1. Results are returned to the caller through the `.dequeue()`-function once they are available. The `CPyrit`-class guarantees that calls to `.enqueue()` and `.dequeue()` correspond in FIFO-order, no matter in which order the hardware actually returned the results. The call to `.dequeue()` can block until the current results are available.
  1. Instances of `Core` call `_gather()` with a desired number of passwords to get work from the queue. The function blocks until unsolved passwords are available on the queue and may return less but not more than the desired number. The calling instance of `Core` is now responsible to call either `_scatter()` to return results or `_revoke()` in case of failure.
  1. Calls to `_gather()` can combine passwords from consecutive calls to `.enqueue()` with matching ESSIDs. The order in which ESSIDs are put on the queue however is preserved towards the hardware to prevent a pipeline-stall towards the caller.

As a hardware-provider you usually don't have anything to do with all this. All you got to do is to add some functionality to `CPyrit`'s `__init__`-function that adds new instances of your `Core`-class to `CPyrit`'s `self.cores`. The following example shows how cores for Nvidia-CUDA are loaded in `CPyrit`'s `__init__`-function:
```
if 'cpyrit._cpyrit_cuda' in sys.modules:
    for dev_idx, device in enumerate(_cpyrit_cuda.listDevices()):
        self.cores.append(CUDACore(queue=self, dev_idx=dev_idx))
        ncpus -= 1
```
Things to note here:
  * `_cpyrit_cuda.listDevices()` uses the underlying hardware-API to get a iterable of available devices.
  * There is one instance of `CUDACore` for every device.
  * We reduce the number of effective CPUs used by CPyrit for every instance of CUDACore. Keeping the GPU-pipeline filled is of much greater interest performance-wise.

This is all.