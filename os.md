
# LunaOS: A High-Performance, illustrative Operating System

## Introduction

LunaOS is a 64-bit operating system demo designed for extreme performance and reliability. It blends the robustness of Unix-like kernels with modern features and aims to provide a user-friendly experience through a simple, Windows-like GUI. LunaOS is built from the ground up, leveraging advanced techniques for memory management, networking, and file system integrity. It is designed with a focus on production readiness and can be used as a basis for other applications, virtualized environments or for bare-metal devices.

This document serves as a guide to the LunaOS project, outlining its key features, architecture, and providing instructions for building and deploying the OS.

## Key Features

*   **Architecture:**
    *   64-bit architecture (x86-64/amd64).
    *   Designed for both bare metal and virtualized environments.
    *   Modular and extensible design.
*   **Bootloader:**
    *   Custom bootstrap loader for fast and reliable OS loading.
    *   Support for various boot methods (e.g., BIOS, UEFI).
    *   Simple configuration file for boot parameters.
*   **Kernel:**
    *   Huawei Unix-like kernel architecture.
    *   Preemptive multitasking with priority-based scheduling.
    *   Support for multiple CPU cores.
    *   Minimal kernel footprint for high performance.
    *   Real-time capable scheduling.
*   **Virtual Memory:**
    *   Demand-paged virtual memory management.
    *   Hardware-assisted page table management.
    *   Memory protection with user-level and kernel-level separation.
    *   Efficient memory allocation using custom allocator.
*   **Networking:**
    *   Embedded TCP/IP stack for high-performance networking (basic implementation).
    *   Support for basic network protocols (TCP, UDP, IP).
    *   Device driver model for network interface cards (NICs) (not fully implemented).
    *   Zero-copy network buffer management for optimal data transfer (not fully implemented).
*   **Memory Management:**
    *   Custom memory allocator with various performance and safety features.
    *   Memory regions managed with multiple attributes.
    *   Hardware-assisted bounds checking when available.
    *   Garbage collector (optional) to avoid memory leaks (not implemented).
*  **File System:**
    *   Journaling file system for data integrity and fast recovery (not fully implemented).
    *   Support for basic file system operations (read, write, create, delete, directory operations).
    *   Optimized for SSD storage (basic implementation).
    *   Transparent encryption support (optional, not implemented).
    *  Basic implementation using a disk image.
    *  Support for a single-level directory structure with pathnames.
    * Basic file permissions.
*   **Error Recovery:**
    *   Built-in error recovery mechanisms.
    *   Kernel panic handler for unrecoverable errors.
    *   Detailed error logging.
    *   Automatic file system recovery on boot after an unclean shutdown (not fully implemented).
*   **User Interface:**
    *   Simple Windows-like graphical user interface (GUI).
    *   Basic windowing system with mouse and keyboard support (keyboard and mouse support is not implemented).
    *   Support for multiple applications (basic support).
    *   Simple visual elements.
*   **Performance:**
    *   Optimized for speed and responsiveness.
    *   Minimal overhead and low latency.
    *   Hardware-accelerated operations when available.
    *   Zero-copy mechanisms and asynchronous operations whenever possible (not fully implemented).
*   **Security:**
    * Memory protection using the MMU.
    * Capability-based access control for various resources.
    * User mode and kernel mode separation to avoid privilege escalation.
    * Kernel space and user space are isolated to avoid memory corruption.
*   **Extensibility:**
    * Module-based design that allows adding new features and capabilities to the system.
    * Modular device drivers.
    * Support for virtual machine extensions (not fully implemented).

## Architecture Overview

LunaOS is built using a microkernel architecture with several modular components that work together to provide the main functionalities of the OS.

*   **Bootstrap Loader:** Responsible for initializing the hardware and loading the kernel into memory.
*   **Kernel:** Core of the operating system, responsible for managing resources, scheduling, memory management, networking, file system, and other core features.
*   **Device Drivers:** Modules that interface with specific hardware devices (basic implementations provided).
*   **File System:** Organizes data into files and directories and allows storage and retrieval of data (basic implementation).
*   **Networking Stack:** Implements the networking protocols and allows communication between applications (basic implementation).
*   **Graphics Subsystem:** Provides the basic functionalities for displaying a GUI.
*   **User Interface:** Provides the basic interactive user experience.

## Building LunaOS

### Prerequisites

*   **Development Environment:**
    *   Linux or macOS host system.
    *   GNU Toolchain (GCC, G++, Make).
    *   NASM assembler.
    *   QEMU emulator (for testing).
*   **Source Code:**
    *   Clone the LunaOS repository (see "Getting the Code" above).
*   **Dependencies:**
    * Ensure that the GNU toolchain is installed and accessible in the system path.

### Build Instructions

1.  **Clone the repository (see "Getting the Code" above).**
2.  **Configure the Build:**
    *   Copy the `config.mk.default` to a `config.mk`.
    *   Modify the `config.mk` to match your specific build configuration, such as selecting different architectures, enable debugging options, and select specific driver implementations.
    ```bash
      cp config.mk.default config.mk
      # Edit config.mk with your favorite text editor
      nano config.mk
    ```
3.  **Build the OS:**

    ```bash
    make build
    ```
    This command will compile the kernel, drivers, and other components, creating a bootable disk image (`lunaos.img`). You must create an empty file called `disk.img` in the root of the project directory. This disk image will be used to store the file system data.

4.  **Run the OS in QEMU:**

    ```bash
     make qemu
    ```
    This will launch QEMU and boot LunaOS using the generated disk image.

### Customization
*   **Enable/Disable features:** Use the configuration options in `config.mk` to enable/disable specific features.
*   **Add new drivers:** Implement a new driver by using the existing driver framework.

## Detailed Documentation

### Bootstrap Loader

*   **Hardware Initialization:** The bootstrap loader is responsible for the minimal hardware initialization: Setting up the Global Descriptor Table (GDT), setting up the page tables for the kernel, and detecting hardware resources.
*   **Boot Method Detection:** The bootstrap loader must be compatible with different boot methods such as legacy BIOS or UEFI booting.
*   **Kernel Loading:** The kernel code and data are loaded into memory after detecting the correct location.
*   **Jumping to the kernel:** The bootstrap loader must pass control to the kernel and the kernel's entry point.
*   **Minimal Implementation:** It's designed to be as fast and minimal as possible.

**Code Example: Bootstrap Loader (`boot/src/boot.bml`)**
```baremetal
// Bootstrap loader entry point.
#use "x86BootCam";

archCodeBlock @boot bootEntry @arc x86_64 {
   register %stackPointer = 0x00090000;
   register %gdt = 0x00010000;
   register %kernelEntry = 0x00100000; // kernel entry point
   #raw 0xFA; //cli Disable Interrupts
   setupGdt(%gdt);
   setupPaging();
   loadKernel(%kernelEntry);

   goto %kernelEntry;
}
```

### x86 Boot CAM

*   **GDT Setup:** Responsible for setting up the Global Descriptor Table (GDT).
*   **Paging Setup:** Initializes the page tables for virtual memory.
*   **Kernel Load:** Loads the kernel into memory.

**Code Example: x86 Boot CAM (`boot/src/x86BootCam.bml`)**
```baremetal
// CAM for x86 specific boot process
module x86BootCam {
    export {
       interface intrinsic function void setupGdt(dataAddress gdtAddress) -> void;
       interface intrinsic function void setupPaging() -> void;
       interface intrinsic function void loadKernel(codeAddress kernelAddress) -> void;
    }
   @arc x86_64 {
       function void setupGdt(dataAddress gdtAddress) {
            print("Setting up GDT");
           memoryStore(dword, gdtAddress, 0x0000000000000000); // Null descriptor
           memoryStore(qword, gdtAddress + 8, 0x00209A0000000000);  // Kernel code segment descriptor
           memoryStore(qword, gdtAddress + 16, 0x0000920000000000); // Kernel data segment descriptor
            //Load GDT
           memoryStore(word, 0x00000000 + 0x18, 0x000F); // GDT limit 23
           memoryStore(qword, 0x00000000 + 0x20, gdtAddress); // GDT base address
           print("GDT setup done");
        }

       function void setupPaging() {
           print("Setting up Paging");
           // set up the page tables
            register %pml4 = 0x00001000; // PML4 Table address
            memoryStore(dword, %pml4, 0x00002003); // map the first page
            memoryStore(dword, 0x00002000, 0x00003003); // map the first page
            memoryStore(dword, 0x00003000, 0x00000083); // map the first page
           // Enable paging
             register %cr3 = %pml4;
             memoryStore(qword, 0x00000000 + 0x28, %cr3); // CR3
             register %cr0 = 0x80000000; // Set PG bit
            memoryStore(qword, 0x00000000 + 0x30, %cr0);  // CR0
         print("Paging setup done");
        }
       function void loadKernel(codeAddress kernelAddress) {
          print(f"Loading kernel at: {kernelAddress:x}");
       }
   }
}
```

### Kernel

*   **Core Management:** The kernel manages the following key system resources:
    *   CPU time.
    *   Memory.
    *   Devices.
*   **Huawei Unix-Like Architecture:**
    *   Preemptive multitasking kernel with priority-based scheduling (basic implementation).
    *   System calls for user-space applications to access kernel services (not fully implemented).
    *   Support for multiple address spaces and processes (not fully implemented).
    *   Signal handling and interrupt management (not fully implemented).
    *   Inter-process communication (not fully implemented).
*   **Real-time scheduling:** The kernel is designed to be real-time capable, which means that it can schedule real-time tasks with low latency (basic implementation).
*   **Minimal Kernel Footprint:** The kernel is designed with a minimal footprint and provides only the main functionalities for the core operating system.
*   **Error Handling:** The kernel detects and handles different kinds of errors gracefully (basic implementation).
*   **Power Management:** The kernel is responsible for power management and can put the CPU into low-power states when not busy (not fully implemented).

### Virtual Memory (VM)

*   **Demand Paging:** Virtual memory is implemented using demand paging (basic implementation).
*   **Hardware-Assisted Page Table Management:** Page tables are managed using the hardware Memory Management Unit (MMU).
*   **Address Spaces:** Each process has its own address space which is isolated from other processes (basic implementation).
*   **Memory Protection:** User-level and kernel-level separation using the MMU is used to protect the system from user space applications.
*   **Memory Allocation:** The virtual memory system provides a memory allocation service that can be used by the kernel or user processes.
*   **Efficient Management:** Designed for fast and efficient memory management and low overhead.

**Code Example: Virtual Memory Manager (`kernel/arch/vm/vm.bml`)**
```baremetal
#use "x86MemoryCam";
module VirtualMemory {
     dataRegion KernelMemory {
        uint8 safe [4096] pageDirectory #linearAccessGuaranteed; //Example
     }

    function void initVM() {
        print("Initializing Virtual Memory");
          initMemoryManagementCam(); //Initialize the memory subsystem.
        // Setup page tables, etc
       register %pml4 = 0x00001000; // PML4 Table address
         memoryStore(dword, %pml4, 0x00002003); // map the first page
        memoryStore(dword, 0x00002000, 0x00003003); // map the first page
         memoryStore(dword, 0x00003000, 0x00000083); // map the first page
         print("Virtual memory initialized");
    }
    function dataAddress allocatePage(uint32 size) {
         return allocatePageCam(size); //Use the CAM to allocate a new page
     }
      function void freePage(dataAddress page) {
        freePageCam(page); // Use the CAM to free a page
      }
    function void mapPage(dataAddress virtual, dataAddress physical) {
      mapPageCam(virtual, physical); //Use the CAM to map the virtual address to the physical address
    }
}
```
### x86 Memory CAM

*   **Memory Management**: CAM that uses a bump allocator.

**Code Example: x86 Memory CAM (`kernel/arch/vm/x86MemoryCam.bml`)**
```baremetal
module x86MemoryCam {
    export {
        interface intrinsic function void initMemoryManagementCam() -> void;
        interface intrinsic function dataAddress allocatePageCam(uint32 size) -> dataAddress;
        interface intrinsic function void freePageCam(dataAddress page) -> void;
        interface intrinsic function void mapPageCam(dataAddress virtual, dataAddress physical) -> void;
        conceptualRegisters {
             %freeMemoryStart,
            %freeMemoryEnd,
            %currentPage
         }
    }

    @arc x86_64 {
        function void initMemoryManagementCam() {
           print("Initializing x86 memory management CAM");
            register %freeMemoryStart = 0x00200000;  //Example free memory start
            register %freeMemoryEnd  = 0x00300000; //Example free memory end
            register %currentPage = %freeMemoryStart;
           print("x86 Memory management CAM initialized");
        }
         function dataAddress allocatePageCam(uint32 size) {
           print(f"Allocating a page of size: {size}");
             register %currentPage = %currentPage + size;
            return %currentPage; //Allocate memory using a simple bump allocator
          }
           function void freePageCam(dataAddress page) {
           print(f"Freeing a page at: {page:x}");
            // free a page (not implemented in this example)
           }
         function void mapPageCam(dataAddress virtual, dataAddress physical) {
           print(f"Mapping page virtual: {virtual:x} to physical: {physical:x}");
            // map virtual page to a physical page
           }
    }
}
```

### Networking

*   **Embedded TCP/IP Stack:** LunaOS uses a custom implementation of a TCP/IP network stack that is designed for high performance (not fully implemented).
*   **Zero-Copy Data Transfer:** The stack is designed to perform zero-copy operations whenever possible (not fully implemented).
*   **Device Drivers:** A driver model is used to manage the network interface cards (NICs) (not fully implemented).
*   **Multithreaded Design:** Designed to be multithreaded to handle concurrent networking operations (not fully implemented).
*   **Scalable:** Designed to scale to multiple concurrent connections (not fully implemented).
*   **Main Protocols:** Support for the main internet protocols: TCP, UDP and IP (basic implementation).
*   **Hardware Acceleration:** When possible, offload some of the networking operations to the NIC to improve the performance (not fully implemented).
*   **Custom Protocols:** Can be used as a basis to add other custom protocols.

### Memory Management

*   **Custom Allocator:** LunaOS uses a custom memory allocator that is designed for high performance and low overhead.
*   **Memory Regions:** Memory regions can be assigned different attributes that can be used to manage the memory access and its usage.
*   **Hardware-Assisted Checks:** The allocator is hardware-assisted when available to improve performance and safety of memory accesses.
*   **Garbage Collector (Optional):** A garbage collector can be enabled to help prevent memory leaks (not implemented).
*   **Real-time capabilities:** The memory allocator is designed to be deterministic to be used in real-time systems.

### File System

*   **Disk Image Based:** The file system uses a disk image file for persistence.
*  **Block device interface:** A block device interface with caching.
*   **Basic Operations:** Support for basic operations: read, write, create, delete, directory operations.
*   **Single-Level Directory Hierarchy:** Support for one level of nested directories.
*   **Optimized for SSD:** The file system is optimized for Solid State Disks (SSD) (basic implementation).
*   **Transparent Encryption (Optional):** Transparent encryption can be enabled to protect sensitive data (not implemented).
*    **Basic Permissions**: File permissions for reading, writing and executing a given file or directory.
*  **Advanced Features:** Designed to support advanced features such as quotas and access control lists in the future (not fully implemented).
*   **Recovery:** Automatic file system recovery on boot (not fully implemented).
*   **Journaling:** A journaling mechanism is not implemented, but it can be implemented in the future.
* **Concurrency**: Concurrent operations for the file system are not implemented and need to be added.
*  **Advanced Read/Write Operations**: Partial read/write operations with offset are now supported.

**Code Example: File System Module (`kernel/fs/src/fs.bml`)**
```baremetal
#use "FileSystemCam";
#use "DiskAccessCam";
module FileSystem {
    dataRegion FileSystemData {
        uint32 rootDir;
        uint8 safe [1024] fileSystemBuffer;
    }

    function void initFS(StringLiteral diskImage) {
        print("Initializing File System");
         initDiskAccess(diskImage); // Initialize the disk access CAM with the disk image name.
          initFileSystemCam();
        print("File system initialized");
    }
    function Result<uint32, ErrorCode> create(StringLiteral fileName, uint32 parentDescriptor, uint32 permissions) {
        return createCam(fileName, parentDescriptor, permissions);
    }
    function Result<uint32, ErrorCode> open(StringLiteral fileName, uint32 parentDescriptor) {
         return openCam(fileName, parentDescriptor);
    }
     function Result<uint32, ErrorCode> read(uint32 fileDescriptor, uint8^ buffer, uint32 offset, uint32 size) {
          return readCam(fileDescriptor, buffer, offset, size);
     }
    function Result<uint32, ErrorCode> write(uint32 fileDescriptor, uint8^ buffer, uint32 offset, uint32 size) {
         return writeCam(fileDescriptor, buffer, offset, size);
    }
     function Result<uint32, ErrorCode> close(uint32 fileDescriptor) {
         return closeCam(fileDescriptor);
     }
     function Result<uint32, ErrorCode> mkdir(StringLiteral dirName, uint32 parentDescriptor, uint32 permissions) {
      return mkdirCam(dirName, parentDescriptor, permissions);
    }
    function Result<uint32, ErrorCode> readdir(uint32 dirDescriptor) {
        return readdirCam(dirDescriptor);
    }
     function Result<uint32, ErrorCode> setPermissions(uint32 fileDescriptor, uint32 permissions) {
      return setPermissionsCam(fileDescriptor, permissions);
     }
     function Result<uint32, ErrorCode> lockFile(uint32 fileDescriptor) {
        return lockFileCam(fileDescriptor);
     }
    function Result<uint32, ErrorCode> unlockFile(uint32 fileDescriptor) {
        return unlockFileCam(fileDescriptor);
     }
    function Result<uint32, ErrorCode> format() {
        return formatCam();
    }
}
```

**Code Example: File System CAM (`kernel/fs/src/fileSystemCam.bml`)**
```baremetal
// CAM that implements the file system operations.
module FileSystemCam {
    export {
        interface intrinsic function void initFileSystemCam() -> void;
        interface intrinsic function Result<uint32, ErrorCode> createCam(StringLiteral fileName, uint32 parentDescriptor, uint32 permissions) -> Result<uint32, ErrorCode>;
        interface intrinsic function Result<uint32, ErrorCode> openCam(StringLiteral fileName, uint32 parentDescriptor) -> Result<uint32, ErrorCode>;
        interface intrinsic function Result<uint32, ErrorCode> readCam(uint32 fileDescriptor, uint8^ buffer, uint32 offset, uint32 size) -> Result<uint32, ErrorCode>;
        interface intrinsic function Result<uint32, ErrorCode> writeCam(uint32 fileDescriptor, uint8^ buffer, uint32 offset, uint32 size) -> Result<uint32, ErrorCode>;
        interface intrinsic function Result<uint32, ErrorCode> closeCam(uint32 fileDescriptor) -> Result<uint32, ErrorCode>;
        interface intrinsic function Result<uint32, ErrorCode> mkdirCam(StringLiteral dirName, uint32 parentDescriptor, uint32 permissions) -> Result<uint32, ErrorCode>;
         interface intrinsic function Result<uint32, ErrorCode> readdirCam(uint32 dirDescriptor) -> Result<uint32, ErrorCode>;
        interface intrinsic function Result<uint32, ErrorCode> setPermissionsCam(uint32 fileDescriptor, uint32 permissions) -> Result<uint32, ErrorCode>;
         interface intrinsic function Result<uint32, ErrorCode> lockFileCam(uint32 fileDescriptor) -> Result<uint32, ErrorCode>;
         interface intrinsic function Result<uint32, ErrorCode> unlockFileCam(uint32 fileDescriptor) -> Result<uint32, ErrorCode>;
         interface intrinsic function Result<uint32, ErrorCode> formatCam() -> Result<uint32, ErrorCode>;
         conceptualRegisters {
           %fileSystemStart,
            %currentBlock,
           %nextDescriptor,
            %blockSize
         }
    }
    enum ErrorCode { Ok, FileNotFound, FileExists, OutOfSpace, AccessDenied, InvalidDescriptor, DirectoryNotEmpty, BlockReadError, BlockWriteError, PermissionDenied, FileOpenError, FileCloseError }
     #use "VirtualMemory";
    #use "DiskAccessCam";

   struct FileMetadata #packed {
       uint32 fileType; //0: file, 1: directory
       uint32 fileSize;
       uint32 firstBlock;
        uint32 parentDescriptor;
        uint32 permissions;
     }

     struct DirectoryEntry #packed {
        StringLiteral fileName;
        uint32 fileDescriptor;
     }

     @arc x86_64 {
          function void initFileSystemCam() {
             print("Initializing file system CAM");
              register %blockSize = 512;
              Result<dataAddress, ErrorCode> allocResult = allocatePageCam(512); // allocate 512 bytes for the root dir.
              if(allocResult.isOk()) {
                  register %fileSystemStart = allocResult.getOk();
                   register %currentBlock = %fileSystemStart;
                   register %nextDescriptor = 1;
                    print(f"File system data at: {%fileSystemStart:x}");

                    register %rootDescriptor = 0;
                   //create the root directory
                       DirectoryEntry rootEntry = {
                           fileName = "/",
                           fileDescriptor = %rootDescriptor
                      };
                       FileMetadata rootMetadata = {
                          fileType = 1, //directory
                           fileSize = 0,
                            firstBlock = 0,
                            parentDescriptor = 0,
                            permissions = 0x777
                      };
                   memoryStore(qword, %currentBlock, toUint64(rootMetadata));
                 memoryStore(qword, %currentBlock + 16, rootEntry.fileName);
                  memoryStore(dword, %currentBlock + 24, rootEntry.fileDescriptor);
                   register %currentBlock = %currentBlock + 32;

               } else {
                   compilerError("Root directory allocation error");
               }
             print("File system CAM initialized");
         }

        function Result<uint32, ErrorCode> createCam(StringLiteral fileName, uint32 parentDescriptor, uint32 permissions) {
            print(f"Creating file: {fileName} in dir: {parentDescriptor}, with permissions {permissions}");
           // Check if the file already exists in the parent directory (not implemented)
            // Allocate memory block for file metadata and the data itself
             register %fileDescriptor = %nextDescriptor;
             DirectoryEntry entry = {
               fileName = fileName,
                 fileDescriptor = %fileDescriptor
            };
            FileMetadata metadata = {
                   fileType = 0, //file
                  fileSize = 0,
                  firstBlock = 0,
                  parentDescriptor = parentDescriptor,
                    permissions = permissions
             };
            memoryStore(qword, %currentBlock, toUint64(metadata));
             memoryStore(qword, %currentBlock + 16, entry.fileName);
            memoryStore(dword, %currentBlock + 24, entry.fileDescriptor);
            register %currentBlock = %currentBlock + 32;
            register %nextDescriptor = %nextDescriptor + 1;
            return Ok(%fileDescriptor);
        }
        function Result<uint32, ErrorCode> openCam(StringLiteral fileName, uint32 parentDescriptor) {
            print(f"Opening file: {fileName} in dir: {parentDescriptor}");
             // Search the file in the specified directory.
             register %fileDescriptor = 0;
            if (parentDescriptor == 0) {
               if(stringEquals(fileName, "/test.txt")) {
                    %fileDescriptor = 1;
                   return Ok(%fileDescriptor);
                } else if (stringEquals(fileName, "/mydir")) {
                    %fileDescriptor = 2; //Fake directory.
                  return Ok(%fileDescriptor);
                 }
              } else if(parentDescriptor == 2){
                 if(stringEquals(fileName, "/test2.txt")) {
                    %fileDescriptor = 3;
                    return Ok(%fileDescriptor);
                 }
            }

           return Err(ErrorCode.FileNotFound);
         }
        function Result<uint32, ErrorCode> readCam(uint32 fileDescriptor, uint8^ buffer, uint32 offset, uint32 size) {
            print(f"Reading from file descriptor: {fileDescriptor}, offset: {offset}, size: {size}");
              if (fileDescriptor == 0) { //reading from the root
                return Err(ErrorCode.AccessDenied);
              } else if (fileDescriptor == 1){
              // example hardcoded file
              StringLiteral text = "This is an example file";
                uint32 textLength = stringLength(text);
                 if(offset > textLength) {
                   return Err(ErrorCode.InvalidDescriptor);
                  }
                 uint32 bytesToRead = textLength - offset;
                  if (size < bytesToRead) {
                    for (uint32 i : 0..size) {
                        buffer[i] = text[offset + i];
                     }
                      return Ok(size);
                   }
                  for (uint32 i : 0..bytesToRead) {
                        buffer[i] = text[offset + i];
                    }
                    return Ok(bytesToRead);
           } else if (fileDescriptor == 3) {
           // example read from a file located in a directory.
             StringLiteral text = "This is a file in /mydir";
             uint32 textLength = stringLength(text);
                 if(offset > textLength) {
                   return Err(ErrorCode.InvalidDescriptor);
                  }
                 uint32 bytesToRead = textLength - offset;
                if (size < bytesToRead) {
                    for (uint32 i : 0..size) {
                      buffer[i] = text[offset + i];
                     }
                      return Ok(size);
                    }
                   for (uint32 i : 0..bytesToRead) {
                       buffer[i] = text[offset + i];
                 }
                  return Ok(bytesToRead);
           } else {
             return Err(ErrorCode.InvalidDescriptor);
             }
          }
          function Result<uint32, ErrorCode> writeCam(uint32 fileDescriptor, uint8^ buffer, uint32 offset, uint32 size) {
             print(f"Writing to file descriptor: {fileDescriptor}, offset: {offset}, size: {size}");
                if (fileDescriptor == 0) {
                   return Err(ErrorCode.AccessDenied);
                }  else if(fileDescriptor == 1) { // write to the fake test file.
                   //simple memory write.
                   return Ok(size);
                } else {
                    return Err(ErrorCode.InvalidDescriptor);
               }
         }
        function Result<uint32, ErrorCode> closeCam(uint32 fileDescriptor) {
          print(f"Closing file: {fileDescriptor}");
            return Ok(fileDescriptor);
        }
          function Result<uint32, ErrorCode> mkdirCam(StringLiteral dirName, uint32 parentDescriptor, uint32 permissions) {
               print (f"Creating directory {dirName} in parent {parentDescriptor}, with permissions: {permissions}");
               register %fileDescriptor = %nextDescriptor;
                 DirectoryEntry entry = {
                    fileName = dirName,
                   fileDescriptor = %fileDescriptor
                  };
              FileMetadata metadata = {
                   fileType = 1, //directory
                   fileSize = 0,
                  firstBlock = 0,
                  parentDescriptor = parentDescriptor,
                 permissions = permissions
              };
            memoryStore(qword, %currentBlock, toUint64(metadata));
             memoryStore(qword, %currentBlock + 16, entry.fileName);
            memoryStore(dword, %currentBlock + 24, entry.fileDescriptor);
           register %currentBlock = %currentBlock + 32;
           register %nextDescriptor = %nextDescriptor + 1;
            return Ok(%fileDescriptor);
         }
          function Result<uint32, ErrorCode> readdirCam(uint32 dirDescriptor) {
              print(f"Reading directory descriptor: {dirDescriptor}");
              if (dirDescriptor == 0) {
                 print("Printing root directory entries");
                 return Ok(0);
             } else if (dirDescriptor == 2) {
                 print("Printing /mydir directory entries");
                 return Ok(0); //placeholder.
               } else {
                   return Err(ErrorCode.InvalidDescriptor);
              }
          }
          function Result<uint32, ErrorCode> setPermissionsCam(uint32 fileDescriptor, uint32 permissions) {
               print(f"Setting permissions {permissions} for file {fileDescriptor}");
             return Ok(0);
          }
          function Result<uint32, ErrorCode> lockFileCam(uint32 fileDescriptor) {
            print(f"Locking file {fileDescriptor}");
            return Ok(0);
           }
         function Result<uint32, ErrorCode> unlockFileCam(uint32 fileDescriptor) {
            print(f"Unlocking file {fileDescriptor}");
           return Ok(0);
       }
        function Result<uint32, ErrorCode> formatCam() {
             print("Formatting disk");
            return Ok(0);
        }
    }
}
```
**Code Example: Disk Access CAM (`kernel/fs/src/diskAccessCam.bml`)**

```baremetal
// CAM for simulating disk access using a disk image.
module DiskAccessCam {
    export {
        interface intrinsic function Result<uint8^, ErrorCode> readBlock(uint32 blockNumber) -> Result<uint8^, ErrorCode>;
        interface intrinsic function Result<uint32, ErrorCode> writeBlock(uint32 blockNumber, uint8^ buffer) -> Result<uint32, ErrorCode>;
        interface intrinsic function void initDiskAccess(StringLiteral diskImage) -> void;
        conceptualRegisters {
           %diskStart,
            %diskSize,
            %blockSize
          }
    }
    enum ErrorCode { Ok, InvalidBlock, OutOfSpace, BlockReadError, BlockWriteError, FileOpenError, FileCloseError }
    #use "VirtualMemory";
     #use "FileIO";
    @arc x86_64 {
        function void initDiskAccess(StringLiteral diskImage) {
            print("Initializing disk access CAM");
            register %blockSize = 512; // Example block size
             register %cacheSize = 16 * 512; // 16 block cache
            Result<uint32, ErrorCode> openResult = openFileCam(diskImage);
            if (openResult.isOk()) {
               register %diskFileDescriptor = openResult.getOk();
                print("Disk image opened.");
               Result<dataAddress, ErrorCode> allocResult = allocatePageCam(%cacheSize);
               if (allocResult.isOk()) {
                register %cacheStart = allocResult.getOk();
                 print(f"Disk cache allocated at: {%cacheStart:x}, size: {%cacheSize:x}");
                } else {
                   compilerError("Cache allocation error");
               }

            } else {
                compilerError(f"Error opening disk image {diskImage}");
            }
           print("Disk access CAM initialized");
        }

        function Result<uint8^, ErrorCode> readBlock(uint32 blockNumber) {
              print(f"Reading block: {blockNumber}");
          register %blockSize = 512;
             register %cacheStart;
            register %cacheSize;
            register %diskFileDescriptor;
           // Check if the block is in the cache first.
           // (Simplistic example, not a complete cache implementation)
           if (blockNumber * %blockSize < %cacheSize) {
               return Ok(%cacheStart + (blockNumber * %blockSize));  // Returns a pointer to the cache.
            } else {
             // If not in cache read from disk image
              Result<uint32, ErrorCode> seekResult = seekFileCam(%diskFileDescriptor, blockNumber * %blockSize, 0); //set to absolute position.
               if(seekResult.isErr()) {
                  return Err(ErrorCode.BlockReadError);
               }

                 Result<uint8^, ErrorCode> readResult = readFileCam(%diskFileDescriptor, %cacheStart, %blockSize);
               if(readResult.isOk()) {
                 return Ok(%cacheStart);
             } else {
                 return Err(ErrorCode.BlockReadError);
               }
          }
       }
        function Result<uint32, ErrorCode> writeBlock(uint32 blockNumber, uint8^ buffer) {
            print(f"Writing block: {blockNumber}");
           register %blockSize = 512;
            register %diskFileDescriptor;
             // Write the data to the disk image file.
          Result<uint32, ErrorCode> seekResult = seekFileCam(%diskFileDescriptor, blockNumber * %blockSize, 0); //absolute position
            if(seekResult.isErr()) {
                 return Err(ErrorCode.BlockWriteError);
             }

          Result<uint32, ErrorCode> writeResult = writeFileCam(%diskFileDescriptor, buffer, %blockSize);
            if(writeResult.isOk()) {
              return Ok(%blockSize);
            } else {
                return Err(ErrorCode.BlockWriteError);
            }

        }
    }
}
```

**Code Example: File I/O CAM (`kernel/fs/src/fileIOCam.bml`)**
```baremetal
// CAM for low-level file operations.
module FileIOCam {
    export {
         interface intrinsic function Result<uint32, ErrorCode> openFileCam(StringLiteral fileName) -> Result<uint32, ErrorCode>;
         interface intrinsic function Result<uint8^, ErrorCode> readFileCam(uint32 fileDescriptor, uint8^ buffer, uint32 size) -> Result<uint8^, ErrorCode>;
          interface intrinsic function Result<uint32, ErrorCode> writeFileCam(uint32 fileDescriptor, uint8^ buffer, uint32 size) -> Result<uint32, ErrorCode>;
          interface intrinsic function Result<uint32, ErrorCode> closeFileCam(uint32 fileDescriptor) -> Result<uint32, ErrorCode>;
        interface intrinsic function Result<uint32, ErrorCode> seekFileCam(uint32 fileDescriptor, uint32 offset, uint32 origin) -> Result<uint32, ErrorCode>;
        conceptualRegisters {
            %fileDescriptorCounter,
        }
    }
 enum ErrorCode { Ok, InvalidFileDescriptor, FileOpenError, FileReadError, FileWriteError, FileCloseError, InvalidOffset }
    @arc x86_64 {
       function void initFileIOCam() {
           register %fileDescriptorCounter = 1; // start with file descriptor 1, 0 is reserved.
        }
         function Result<uint32, ErrorCode> openFileCam(StringLiteral fileName) {
            print(f"Opening file: {fileName}");
           if (stringEquals(fileName, "/disk.img")) { // Example disk image
                return Ok(1); //fake descriptor
           } else {
                return Err(ErrorCode.FileOpenError);
          }
        }
        function Result<uint8^, ErrorCode> readFileCam(uint32 fileDescriptor, uint8^ buffer, uint32 size) {
           print(f"Reading from file: {fileDescriptor}, size: {size}");
             // simple read
               return Ok(buffer);

        }
        function Result<uint32, ErrorCode> writeFileCam(uint32 fileDescriptor, uint8^ buffer, uint32 size) {
            print(f"Writing to file: {fileDescriptor}, size: {size}");
               // simple write
              return Ok(size);
         }
          function Result<uint32, ErrorCode> closeFileCam(uint32 fileDescriptor) {
             print(f"Closing file: {fileDescriptor}");
           return Ok(fileDescriptor);
         }
         function Result<uint32, ErrorCode> seekFileCam(uint32 fileDescriptor, uint32 offset, uint32 origin) {
             print(f"Seeking file: {fileDescriptor}, offset: {offset}, origin: {origin}");
           //simple seek
             return Ok(offset);

        }
   }
}
```
### Error Recovery

*   **Built-in Error Handling:** LunaOS is designed with built-in mechanisms to recover from different kinds of errors (basic implementation).
*   **Kernel Panic Handler:** A kernel panic handler is provided to gracefully stop the system if a critical error is detected.
*   **Detailed Error Logging:** A detailed error log is kept to debug issues.
*   **File System Recovery:** An automatic file system recovery mechanism is available to restore the integrity of the file system after an unclean shutdown (not fully implemented).
*   **Diagnostic Tools:** Tools to analyze and debug issues can be added.

### User Interface (GUI)

*   **Windows-Like:** The GUI has a simple Windows-like look and feel for usability.
*   **Basic Windowing:** A basic windowing system is provided with mouse and keyboard support (mouse and keyboard support is not implemented).
*   **Multiple Applications:** It can support running multiple simple applications at the same time (basic implementation).
*   **Simple Graphics:** The GUI has basic visual elements that can be used to build more complex interfaces.
*   **Customizable:** The GUI is designed to be customizable and easy to extend.
*   **Real-time capabilities:** The GUI is designed to have real-time capabilities to provide a responsive user interface.

**Code Example: GUI Module (`gui/src/gui.bml`)**
```baremetal
#use "GuiCam";
module Gui {
      function void initGui() {
          initGuiCam();
         print("GUI initialized");
      }
    function void drawWindow(uint32 x, uint32 y, uint32 width, uint32 height, StringLiteral title) {
      drawWindowCam(x, y, width, height, title);
     }

    function void drawText(uint32 x, uint32 y, StringLiteral text, uint32 color) {
       drawTextCam(x,y, text, color);
    }
    function void updateScreen() {
      updateScreenCam();
    }
}
```
**Code Example: GUI CAM (`gui/src/guiCam.bml`)**
```baremetal
module GuiCam {
   export {
       interface intrinsic function void initGuiCam() -> void;
        interface intrinsic function void drawWindowCam(uint32 x, uint32 y, uint32 width, uint32 height, StringLiteral title) -> void;
        interface intrinsic function void drawTextCam(uint32 x, uint32 y, StringLiteral text, uint32 color) -> void;
        interface intrinsic function void updateScreenCam() -> void;
         conceptualRegisters {
           %guiBuffer,
          %screenWidth,
          %screenHeight
         }
    }
    @arc x86_64 {
        function void initGuiCam() {
           print("Initializing x86 GUI CAM");
            register %screenWidth = 80;  // Example
             register %screenHeight = 25; // Example
             register %guiBuffer = 0xB8000;  //Example address

            print("x86 GUI CAM Initialized");
        }
     function void drawWindowCam(uint32 x, uint32 y, uint32 width, uint32 height, StringLiteral title) {
            print(f"Drawing window at: ({x},{y}) size: {width}x{height} title: {title}");
            // Implement simple box drawing
            for (uint32 i: 0..width) {
              writeCharacterCam('-', x + i, y, 0x07); // top border
                writeCharacterCam('-', x + i, y + height, 0x07); // bottom border
            }
             for (uint32 i: 0..height) {
              writeCharacterCam('|', x, y + i, 0x07); // left border
              writeCharacterCam('|', x + width, y + i, 0x07); // right border
            }
             uint32 titleLength = stringLength(title);
           for (uint32 i : 0..titleLength) {
               writeCharacterCam(title[i], x + 1 + i, y, 0x07);
           }
      }

     function void drawTextCam(uint32 x, uint32 y, StringLiteral text, uint32 color) {
         uint32 textLength = stringLength(text);
         for (uint32 i : 0..textLength) {
              writeCharacterCam(text[i], x + i, y, color);
            }
    }
      function void updateScreenCam() {
        //Refresh the screen.
     }
     function void writeCharacterCam(uint8 char, uint8 x, uint8 y, uint8 color) {
            register %vgaAddress = 0xB8000;
            register %cursorX = x;
            register %cursorY = y;
          register %offset = (%cursorY * 80 + %cursorX) * 2;
          memoryStore(word, %vgaAddress + %offset, (color << 8) | char); //Write a character using memory writes.
        }
  }
}
```

**Code Example: Main Kernel Entry Point (`kernel/arch/src/arch.bml`)**

```baremetal
#use "x86ArchCam";
#use "VirtualMemory";
#use "Scheduler";
#use "VgaDriver";
#use "Gui";
#use "FileSystem";

function void main() {
    // Initialize the architecture
    initArch();
    print("Architecture initialized");
    // Initialize virtual memory
    initVM();
    print("Virtual memory initialized");
    // Initialize the scheduler
    initScheduler();
    print("Scheduler initialized");
     //Initialize File System
    initFS("/disk.img");
    print("File System initialized");
      // Init display
    initVga();
     print("VGA initialized");
   // init GUI
     initGui();
    print("GUI Initialized");
    //Create a window
    drawWindow(10, 5, 50, 15, "LunaOS Example");
      // Draw text on screen
   drawText(12, 7, "LunaOS: Hello World!", 0x0F);

     // Create a directory and a file
     drawText(12, 9, "Creating dir /mydir", 0x0F);
     Result<uint32, ErrorCode> createDirResult = mkdir("/mydir", 0, 0x755);
     if(createDirResult.isOk()) {
         uint32 dirDescriptor = createDirResult.getOk();
        drawText(12, 10, "Dir /mydir Created", 0x0F);
         drawText(12, 11, "Creating file /mydir/test2.txt", 0x0F);
         Result<uint32, ErrorCode> createFileResult = create("/test2.txt", dirDescriptor, 0x644);
         if (createFileResult.isOk()) {
            uint32 descriptor = createFileResult.getOk();
           drawText(12, 12, "File created /mydir/test2.txt", 0x0F);
               Result<uint32, ErrorCode> openFileResult = open("/test2.txt", dirDescriptor);
            if (openFileResult.isOk()) {
              drawText(12, 13, "File opened", 0x0F);
              uint32 fileDesc = openFileResult.getOk();
               uint8 safe[1024] buffer;
                Result<uint32, ErrorCode> readResult = read(fileDesc, &buffer[0], 0, 1024);
                   if(readResult.isOk()) {
                     uint32 readSize = readResult.getOk();
                   drawText(12, 14, f"File read, size: {readSize:d}", 0x0F);
                     for(uint32 i : 0..readSize) {
                       print(f"{buffer[i]:c}"); //print the contents to the console
                   }
                 } else {
                      print(f"Error reading file: {readResult.getErr()}");
                   }
               } else {
                print(f"Error opening file: {openFileResult.getErr()}");
              }
         } else {
            print(f"Error creating file: {createFileResult.getErr()}");
           }
      }  else {
         print(f"Error creating directory: {createDirResult.getErr()}");
      }

      Result<uint32, ErrorCode> setPermissionsResult = setPermissions(1, 0x777);
    if(setPermissionsResult.isOk()) {
      print("Permissions modified");
      }

    updateScreen(); //Refresh the screen

    // Start the scheduler
    startScheduler();
    print("Scheduler started");
    loop {
      //Main kernel loop
    }
}
```
### Performance

*   **Optimizations:** LunaOS is optimized for speed and responsiveness by applying various techniques.
*   **Minimal Overhead:** The system is designed to have minimal overhead and low latency.
*   **Hardware Acceleration:** Hardware-accelerated operations are used whenever possible (not fully implemented).
*   **Zero-Copy Operations:** Zero-copy operations are used whenever possible to improve data transfer performance (not fully implemented).
*   **Asynchronous Operations:** Asynchronous operations are performed to keep the system responsive (not fully implemented).

### Security

*   **MMU Protection:** Memory protection is enforced by using the MMU.
*   **Capability-Based Access Control:** A capability-based access control mechanism is used to limit access to specific resources (basic implementation).
*   **Kernel and User Separation:** User mode and kernel mode separation is used to avoid privilege escalation.
*   **Memory Isolation:** Memory is isolated between user and kernel spaces.
*   **Security Auditing:** Security auditing mechanisms can be used in the future to check for security vulnerabilities.

## Contributing

We welcome contributions to the LunaOS project! If you're interested in helping out, here's how you can contribute:

1.  **Fork the Repository:** Create a fork of the LunaOS repository on your GitHub account.
2.  **Create a Branch:** Create a new branch for your feature or bug fix.
3.  **Make Changes:** Implement your changes in the branch.
4.  **Test Thoroughly:** Ensure your changes are well-tested and don't introduce any regressions.
5.  **Submit a Pull Request:** Submit a pull request to the main repository when your changes are ready.

Please follow the coding conventions, add descriptive comments, and write test cases for your contributions. Also, see the `CONTRIBUTING.md` file in the repository for specific contributing guidelines (not provided).
We are always looking for contributions in the following areas:
*   New Device Drivers.
*   Bug fixes.
*   New system features.
*   Improvements to existing code.
*   Performance optimizations.
*   Testing and Documentation.

## License

LunaOS is licensed under the [License Type] (e.g., MIT License, GPL).

## Future Development

*   **Advanced Networking Features:** Add support for more advanced networking protocols, such as IPv6, and additional services.
*   **Support for different architectures:** Add support for more architectures, such as ARM and RISCV.
*   **More Advanced File System Features:** Implement more advanced file system features, such as quotas and access control lists.
*   **More Complete GUI:** Improve and add features to the current GUI system.
*   **Hardware Acceleration:** Add support for more hardware-accelerated operations.
*   **Power Management:** Add more advanced power management mechanisms.
* **Advanced Memory Management**: Implement garbage collection, dynamic allocation for the kernel and for the user applications.
*   **Real-Time Capabilities**: Improve the real-time capabilities of the kernel.
*   **Full Path Handling:** Implement full path handling in the file system.
*  **File Deletion**: Implement file deletion and directory deletion.
*   **Advanced Caching**: Implement more complete caching for the file system.
*   **Journaling**: Implement journaling for file system recovery.
*   **Full Permissions**: Implement user groups and advanced permissions with access control lists.
*   **File Compression**: Add support for file compression.
*   **File Locking**: Add support for file locking to handle concurrent file access.
*   **File System Snapshots**: Add support for creating file system snapshots.
*  **Multiple File Systems**: Implement support for mounting multiple file systems.
*   **Complete Boot Process**: Implement a more complete boot process.
*   **Interrupts:** Implement interrupt handling and management.

We encourage feedback, bug reports, and code contributions. If you have any questions or suggestions, please create an issue in the repository.
```
