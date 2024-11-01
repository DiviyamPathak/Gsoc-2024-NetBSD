# Gsoc-2024-NetBSD
# Setroot:
```mermaid 
flowchart TD  
    A[setroot Start] --> B{md_is_root?}  
    B -->|Yes| C[setroot_md]  
    B -->|No| D{TFTPROOT defined?}  
      
    D -->|Yes| E[tftproot_dhcpboot]  
    E -->|Success| F[setroot_md]  
    D -->|No| G[setroot_nfs]  
    E -->|Failure| G  
      
    G --> H{rootspec == NULL & bootdv == NULL?}  
    H -->|Yes| I[Set RB_ASKNAME]  
    H -->|No| J  
      
    I --> J{Root Device Loop}  
    J -->|RB_ASKNAME set| K[setroot_ask]  
    J -->|RB_ASKNAME not set| L[setroot_root]  
      
    L -->|root_device == NULL| M{Wait timeout?}  
    M -->|No| N[Sleep 1 sec]  
    M -->|Yes| O[Set RB_ASKNAME]  
    N --> L  
    O --> J  
      
    K --> P[Configure root_device]  
    L -->|root_device found| P  
      
    P --> Q[setroot_dump]  
    Q --> R[End]

```
During a system boot up, based on kernel Config file, system mounts root filesystem. This is handled by setroot function in kern_subr.c. 
device and partition are defined with syntax 
```
config netbsd root on XXX_device type XXX 
```
device name with partition is loaded onto rootspec variable which is used by setroot and associated functions to select a device. device can also be a network device 
in that case a valid interface name should be specified. if instead of a device ' ? ' is specified, setroot uses boot device(from bootspec) as root device.
Boot device is set by findroot function. Findroot is machine dependent. x86/x86_autoconf.c contains function which interact with BIOS entries to configure boot spec variable.
it also has different cases for different device type [Christoph badura's documentation](https://hackmd.io/qS2qYmBNQ9-_C_4gBBzDNQ). 


1. Primary Functions:  
* `setroot()`: Main entry point that orchestrates root device selection  
* `setroot_nfs()`: Handles NFS root configuration  
* `setroot_md()`: Sets up memory disk as root device  
* `setroot_ask()`: Interactive root device configuration  
* `setroot_root()`: core root device configuration  
* `setroot_dump()`: Configures dump device  
2. Key Control Flow:  
   a) Initial Setup:  
      * Checks if config has specified rootdev and partition (through rootspec), otherwise uses dev and partition system booted from ( bootspec)  
      * Verifies if memory disk should be root (md\_is\_root)  
      * Handles TFTPROOT configuration if enabled  

    b) Device Selection:    
     * If no boot device and no root specified, enters interactive mode  
     * Waits up to ROOT\_WAITTIME (default 20 seconds) for device availability  
     * Falls back to interactive mode i.e. asks user for root device and dump device if device doesn't appear

    c) Interactive Configuration (`setroot_ask`):  
     * Prompts for root device  
     * Prompts for dump device  
     * Prompts for filesystem type  
     * Supports special commands (halt, reboot, ddb) debugger is only supported if src was built with debugger option  
3. Supported features types:  
  a) Device Support:  
    * Handles disk devices with partitions  
    * Supports network interfaces  
    * Supports memory disk devices  
    * Supports disk wedges
   
    b) Filesystem Support:  
      * Generic filesystem support  
      * NFS support  
      * Allows custom filesystem type selection
       
    c) Dump Device Configuration:  
    * Automatic selection based on root device  
    * Manual configuration support  
    * Support for "none" as dump device  
4. Important Variables:  
  * `rootdev`: Device number for root device  
  * `dumpdev`: Device number for dump device  
  * `dumpcdev`: Character device for dumps  
  * `root_device`: Device structure for root device  
  * `booted_device`: Device we booted from   
  * `rootspec`: Specification for root device from config file   
  * `bootspec`: Boot device specification  
5. Special Handling:    
a) NFS Root:
   - Automatically selects the network interface if needed.
   - Skips partition handling for network devices.    
     
    b) Memory Disk:
     - Special handling when `md_is_root` is set.
     - Creates `md0` device if needed.

    c) Dump Device Selection**:
     - Uses partition 'b' by default on the root disk.
     - Allows "none" as a valid option.
     - Special handling for network root devices.

6. Error Handling:  
  * Timeout mechanism for device availability  
  * Fallback to interactive mode on failures  
  * Validation of device and filesystem selections  
  * Clear error messages for invalid selections

# Setroot\_md and md is disk

# Tftproot\_dhcp

# Setroot\_root

### **Function: `setroot_root`**

#### **Purpose**

The `setroot_root` function is responsible for configuring the root device of the system during the boot process. It selects the appropriate root device based on a specified configuration (`rootspec`) or falls back to the boot device if `rootspec` is not defined. This function also determines the major device number and partition of the root device, ensuring that the system can mount the root filesystem correctly.

#### **Parameters**

* **`device_t bootdv`**: Represents the device from which the system was booted. It serves as the default root device if no specific root device is provided by `rootspec`.  
* **`int bootpartition`**: Specifies the partition number of the boot device. Used when the root device is a disk with partitions.

#### **Internal Variables**

* **`device_t rootdv`**: The selected root device, determined either from `rootspec` or defaulted to the boot device (`bootdv`).  
* **`int majdev`**: Holds the major device number of the root device, which is used to identify the device type in the system.  
* **`const char *rootdevname`**: Name of the root device based on the major number, used in error messages or device identification.  
* **`char buf[128]`**: Buffer to store the root device name, used for formatted messages.  
* **`dev_t nrootdev`**: Represents the device number parsed from `rootspec`.

#### **Function Workflow**

1. **Checking for Root Device Specification (`rootspec`)**:  
   * If `rootspec` is `NULL`, the function defaults to using `bootdv` as the root device.  
   * If `bootdv` is defined, `majdev` is assigned the major number of `bootdv`.  
   * For disk devices, it calculates the root device ID (`rootdev`) using `MAKEDISKDEV` if the device uses partitions, or `makedev` otherwise.  
2. **Parsing the `rootspec`**:  
   * If `rootspec` is defined, `parsedisk` attempts to parse it and assign the resulting device to `rootdv`.  
   * If parsing fails, `rootdev` and `rootdv` are computed from the major device name (`rootdevname`) and device unit.  
   * The function then calls `finddevice` to locate `rootdv` based on the constructed device name in `buf`.  
3. **Error Handling**:  
   * If `majdev` is invalid or `rootdv` is `NULL`, error messages are printed, and the function exits early.  
4. **Final Root Device Determination**:  
   * After successfully identifying `rootdv`, the function checks the device class of `rootdv` to ensure it is either a network interface (`DV_IFNET`) or a disk (`DV_DISK`).  
   * It then displays the name of the root device and appends the partition letter if the device supports partitions.  
5. **Setting Global Root Device**:  
   * The global `root_device` is assigned to `rootdv`, and `setroot_dump` is called to complete the root device setup.

#### **Notes**

* **Device and Partition Handling**: The function is designed to handle both partitioned and non-partitioned devices gracefully.  
* **Error Reporting**: Provides error messages when the root device cannot be determined.  
* **Flexibility**: The function supports dynamic root device specification via `rootspec`, making it adaptable to various boot configurations.

# 
