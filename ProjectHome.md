<a href='http://flattr.com/thing/104890/GPU-accelerated-attack-against-WPA-PSK-authentication'>
<img src='http://api.flattr.com/button/flattr-badge-large.png' alt='Flattr this' border='0' title='Flattr this' /></a>

**_Pyrit_** allows to create massive databases, pre-computing part of the [IEEE 802.11 WPA/WPA2-PSK](https://secure.wikimedia.org/wikipedia/en/wiki/Wi-Fi_Protected_Access) authentication phase in a space-time-tradeoff. Exploiting the computational power of Many-Core- and other platforms through [ATI-Stream](http://ati.amd.com/technology/streamcomputing/), [Nvidia CUDA](http://www.nvidia.com/object/cuda_home.html) and [OpenCL](http://www.khronos.org/opencl/), it is currently by far the most powerful attack against one of the world's most used security-protocols.


WPA/WPA2-PSK is a subset of [IEEE 802.11 WPA/WPA2](https://secure.wikimedia.org/wikipedia/en/wiki/Wi-Fi_Protected_Access) that skips the complex task of key distribution and client authentication by assigning every participating party the same _pre shared key_. This _master key_ is derived from a password which the administrating user has to pre-configure e.g. on his laptop and the Access Point. When the laptop creates a connection to the Access Point, a new _session key_ is derived from the _master key_ to encrypt and authenticate following traffic. The "shortcut" of using a single _master key_ instead of _per-user keys_ eases deployment of WPA/WPA2-protected networks for home- and small-office-use at the cost of making the protocol vulnerable to brute-force-attacks against it's key negotiation phase; it allows to ultimately reveal the password that protects the network. This vulnerability has to be considered exceptionally disastrous as the protocol allows much of the key derivation to be pre-computed, making simple brute-force-attacks even more alluring to the attacker.
For more background see [this article](http://pyrit.wordpress.com/the-twilight-of-wi-fi-protected-access/) on the project's [blog](http://pyrit.wordpress.com).

The author does not encourage or support using _Pyrit_ for the infringement of peoples' communication-privacy. The exploration and realization of the technology discussed here motivate as a purpose of their own; this is documented by the open development, strictly sourcecode-based distribution and 'copyleft'-licensing.

_Pyrit_ is free software - free as in freedom. Everyone can inspect, copy or modify it and share derived work under the GNU General Public License v3+. It compiles and executes on a wide variety of platforms including FreeBSD, MacOS X and Linux as operation-system and x86-, alpha-, arm-, hppa-, mips-, powerpc-, s390 and sparc-processors.



Attacking WPA/WPA2 by brute-force boils down to to computing _Pairwise Master Keys_ as fast as possible. Every _Pairwise Master Key_ is 'worth' exactly one megabyte of data getting pushed through [PBKDF2](http://en.wikipedia.org/wiki/PBKDF2)-[HMAC](http://en.wikipedia.org/wiki/Hmac)-[SHA1](http://en.wikipedia.org/wiki/SHA_hash_functions). In turn, computing 10.000 PMKs per second is equivalent to hashing 9,8 gigabyte of data with [SHA1](http://en.wikipedia.org/wiki/SHA_hash_functions) in one second. The following graph shows various performance numbers measured on platforms supported by Pyrit.

![http://pyrit.googlecode.com/svn/tags/opt/pyritperfaa3.png](http://pyrit.googlecode.com/svn/tags/opt/pyritperfaa3.png)


The following graph shows an example of multiple computational nodes accessing a single storage server over various ways provided by Pyrit:

  * A single storage (e.g. a MySQL-server)
  * A local network that can access the storage-server directly and provide four computational nodes on various levels with only one node actually accessing the storage server itself.
  * Another, untrusted network can access the storage through Pyrit's RPC-interface and provides three computional nodes, two of which actually access the RPC-interface.

![http://pyrit.googlecode.com/svn/tags/opt/deployment_example.png](http://pyrit.googlecode.com/svn/tags/opt/deployment_example.png)

# What's new #

See http://pyrit.wordpress.com


# How to use #

_Pyrit_ compiles and runs fine on Linux, MacOS X and BSD. I don't care about Windows; drop me a line (read: patch) if you make _Pyrit_ work without copying half of GNU ...

A guide for [installing](Installation.md) _Pyrit_ on your system can be found in the [wiki](http://code.google.com/p/pyrit/w/list). There is also a [Tutorial](Tutorial.md) and a [reference manual](ReferenceManual.md) for the commandline-client.



# How to participate #

You may want to read [this wiki-entry](ExtendPyrit.md) if interested in porting Pyrit to new hardware-platform.
Contributions or bug reports should be posted on the [Issue-tracker](http://code.google.com/p/pyrit/issues/list). General questions get answered on Pyrit's mailing-list; just send an eMail to pyrit@googlegroups.com or see http://groups.google.com/group/pyrit.