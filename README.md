# Gsoc 2024 NetBSD 
## Project Test root device and root file system selection
### Project Details 
In NetBSD's later stage of bootup, Kernel finds device with root filesystem. this search is assisted with the help of config file of kernel.
NetBSD has Testing suite called ATF (Automated testing framework) But this suite does not cover this area of function. We attempt to write test for root device and filesystem selection.
Functions we need to test are setroot , it utilizes number of other functions for different scenarios which are documented in my [documentation]() . we will use vnd to use device simulation in userspace. 
### Project Goals
our goals could be divided into 2 parts 
1. Documentation
2. Writting ATF Tests

we had to document config file's constructs for device specifications and types. Functionality and control flow of root device and file system selection. We also had to take note of all the edge cases for which we could make test cases. There are number of global variables that interact with setroot inorder for a final root_device variable to be set. We had to document them and point at which it would change to produce specific scenarios. 
Finally We have to write ATF based Unit tests for all cases using rump kernels.

### curent state and future forward
We have to move the Tests so that it uses rump kernel and also rewrite/refactor part of kern_subr.c for code quality. 

### Code Details
here is my tree for project [/src](https://github.com/DiviyamPathak/src/tree/gsoc-setroot-nb10)

This function handles wildcard entry in config file else handles specified device [Test Cases for setroot_root](https://github.com/DiviyamPathak/src/commit/8c8bddfacf986a31f2ab812fc250f454437bbff6)

this function handles network booting[Test Cases for tftproot_dhcpboot](https://github.com/DiviyamPathak/src/commit/b85d451366436fc3e0647972b484daa38e268d6c)

currently they are these functions run in userspace hence calls mocked functions while running, we intend to move them to use rumpkernel. we will use vnd(8) for setting up neccessary 
enviroment for running without mocking internal functions. 
Global Varibles are set outside of kernel space manually.


### Documentation 
Here is Documentation for all functions in kern_subr.c 
[Docs](./setrootDocumentation.md)

it covers main setroot() and following
  - setroot_root()
  - setroot_ask()
  - tftproot_dhcpboot()
  - setroot_md() 
these functions perform root device and fs selection operation. they cover a wide range of scenarios.
there are also global variables, I will cal them intermediatries between config and kern_subr.c.  
