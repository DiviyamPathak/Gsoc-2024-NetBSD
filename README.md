# Gsoc 2024 NetBSD 
## Project Test root device and root file system selection
### Project Details 
In NetBSD's later stage of bootup, Kernel finds device with root filesystem. this search is assisted with the help of config file of kernel.
NetBSD has Testing suite called ATF (Automated testing framework) But this suite does not cover this area of function. We attempt to write test for root device and filesystem selection.
Functions we need to test are setroot , it utilizes number of other functions for different scenarios which are documented in my [documentation]() . we will use vnd to use device simulation in userspace. 

### Code Details
here is my tree for project [/src](https://github.com/DiviyamPathak/src/tree/gsoc-setroot-nb10)

[Test Cases for setroot_root](https://github.com/DiviyamPathak/src/commit/8c8bddfacf986a31f2ab812fc250f454437bbff6)

[Test Cases for tftproot_dhcpboot](https://github.com/DiviyamPathak/src/commit/b85d451366436fc3e0647972b484daa38e268d6c)

### Documentation 
Here is Documentation for all functions in kern_subr.c 
[Docs](./setrootDocumentation.md)
