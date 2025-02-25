# Reducing latency of computations
This FPGA tutorial demonstrates how to use the `use_stall_enable_clusters` attribute to reduce the area and latency of your FPGA kernels.  This attribute may reduce the FPGA FMax for your kernels.

| Optimized for                     | Description
|:---                               |:---
| OS                                | Linux* Ubuntu* 18.04/20.04 <br> RHEL*/CentOS* 8 <br> SUSE* 15 <br> Windows* 10
| Hardware                          | Intel&reg; Programmable Acceleration Card (PAC) with Intel Arria&reg; 10 GX FPGA <br> Intel&reg; FPGA Programmable Acceleration Card (PAC) D5005 (with Intel Stratix&reg; 10 SX) <br> Intel&reg; FPGA 3rd party / custom platforms with oneAPI support <br> **Note**: Intel&reg; FPGA PAC hardware is only compatible with Ubuntu 18.04*
| Software                          | Intel&reg; oneAPI DPC++ Compiler <br> Intel&reg; FPGA Add-On for oneAPI Base Toolkit
| What you will learn               |  What the `use_stall_enable_clusters` attribute does <br> How `use_stall_enable_clusters` attribute affects resource usage and latency <br> How to apply the `use_stall_enable_clusters` attribute to kernels in your program
| Time to complete                  | 15 minutes (emulator and report) <br> several hours (fpga bitstream generation)

## Purpose
The `use_stall_enable_clusters` attribute enables you to direct the compiler to reduce the area and latency of your kernel.  Reducing the latency will not have a large effect on loops that are pipelined, unless the number of iterations of the loop is very small.
Computations in an FPGA kernel are normally grouped into *Stall Free Clusters*. This allows simplification of the signals within the cluster, but there is a FIFO queue at the end of the cluster that is used to save intermediate results if the computation needs to stall. *Stall Enable Clusters* save area and cycles by removing the FIFO queue and passing the stall signals to each part of the computation.  These extra signals may cause the FMax to be reduced

> **Note**: If you specify `[[intel::use_stall_enable_clusters]]` on one or more kernels, this may reduce the FMax of the generated FPGA bitstream, which may reduce performance on all kernels.

> **Note**: The `use_stall_enable_clusters` attribute is not applicable for designs that target the Stratix&reg; 10 architecture unless the `-Xshyper-optimized-handshaking=off` argument is passed to `dpcpp`
### Example: Using the `use_stall_enable_clusters` attribute
```
h.single_task<class KernelComputeStallFree>( [=]() [[intel::use_stall_enable_clusters]] {
  // The computations in this device kernel will use Stall Enable clusters
  Work(accessor_vec_a, accessor_vec_b, accessor_res);
});
```
The FPGA compiler will use *Stall Enable Clusters* for the kernel when possible.  Some computations may not be able to stall and need to be placed in a *Stall Free Cluster*.

### Additional Documentation
- [Explore SYCL* Through Intel&reg; FPGA Code Samples](https://software.intel.com/content/www/us/en/develop/articles/explore-dpcpp-through-intel-fpga-code-samples.html) helps you to navigate the samples and build your knowledge of FPGAs and SYCL.
- [FPGA Optimization Guide for Intel&reg; oneAPI Toolkits](https://software.intel.com/content/www/us/en/develop/documentation/oneapi-fpga-optimization-guide) helps you understand how to target FPGAs using SYCL and Intel&reg; oneAPI Toolkits.
- [Intel&reg; oneAPI Programming Guide](https://software.intel.com/en-us/oneapi-programming-guide) helps you understand target-independent, SYCL-compliant programming using Intel&reg; oneAPI Toolkits.

## Key Concepts
* Description of the `use_stall_enable_clusters` attribute
* How `use_stall_enable_clusters` attribute affects resource usage and loop throughput
* How to apply the `use_stall_enable_clusters` attribute to kernels in your program


## Building the `stall_enable` Tutorial
> **Note**: If you have not already done so, set up your CLI
> environment by sourcing  the `setvars` script located in
> the root of your oneAPI installation.
>
> Linux*:
> - For system wide installations: `. /opt/intel/oneapi/setvars.sh`
> - For private installations: `. ~/intel/oneapi/setvars.sh`
>
> Windows*:
> - `C:\Program Files(x86)\Intel\oneAPI\setvars.bat`
>
>For more information on environment variables, see **Use the setvars Script** for [Linux or macOS](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-linux-or-macos.html), or [Windows](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-windows.html).

### Include Files
The included header `dpc_common.hpp` is located at `%ONEAPI_ROOT%\dev-utilities\latest\include` on your development system.

### Running Samples in Intel&reg; DevCloud
If running a sample in the Intel&reg; DevCloud, remember that you must specify the type of compute node and whether to run in batch or interactive mode. Compiles to FPGA are only supported on fpga_compile nodes. Executing programs on FPGA hardware is only supported on fpga_runtime nodes of the appropriate type, such as fpga_runtime:arria10 or fpga_runtime:stratix10.  Neither compiling nor executing programs on FPGA hardware are supported on the login nodes. For more information, see the Intel&reg; oneAPI Base Toolkit Get Started Guide ([https://devcloud.intel.com/oneapi/documentation/base-toolkit/](https://devcloud.intel.com/oneapi/documentation/base-toolkit/)).

When compiling for FPGA hardware, it is recommended to increase the job timeout to 12h.


### Using Visual Studio Code*  (Optional)

You can use Visual Studio Code (VS Code) extensions to set your environment, create launch configurations,
and browse and download samples.

The basic steps to build and run a sample using VS Code include:
 - Download a sample using the extension **Code Sample Browser for Intel&reg; oneAPI Toolkits**.
 - Configure the oneAPI environment with the extension **Environment Configurator for Intel&reg; oneAPI Toolkits**.
 - Open a Terminal in VS Code (**Terminal>New Terminal**).
 - Run the sample in the VS Code terminal using the instructions below.
 - (Linux only) Debug your GPU application with GDB for Intel&reg; oneAPI toolkits using the **Generate Launch Configurations** extension.

To learn more about the extensions, see the
[Using Visual Studio Code with Intel&reg; oneAPI Toolkits User Guide](https://www.intel.com/content/www/us/en/develop/documentation/using-vs-code-with-intel-oneapi/top.html).

### On a Linux* System

1. Generate the `Makefile` by running `cmake`.
  ```
  mkdir build
  cd build
  ```
  To compile for the Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA, run `cmake` using the command:
  ```
  cmake ..
  ```
   Alternatively, to compile for the Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX), with hyper optimized handshaking disabled, run `cmake` using the command:

   ```
   cmake .. -DFPGA_BOARD=intel_s10sx_pac:pac_s10 -DNO_HYPER_OPTIMIZATION=true
   ```
   You can also compile for a custom FPGA platform. Ensure that the board support package is installed on your system. Then run `cmake` using the command:
   ```
   cmake .. -DFPGA_BOARD=<board-support-package>:<board-variant>
   ```

2. Compile the design through the generated `Makefile`. The following build targets are provided, matching the recommended development flow:

   * Compile for emulation (fast compile time, targets emulated FPGA device):
      ```
      make fpga_emu
      ```
   * Generate the optimization reports:
     ```
     make report
     ```
   * Compile for FPGA hardware (longer compile time, targets FPGA device):
     ```
     make fpga
     ```
3. (Optional) As the above hardware compile may take several hours to complete, FPGA precompiled binaries (compatible with Linux* Ubuntu* 18.04) can be downloaded <a href="https://iotdk.intel.com/fpga-precompiled-binaries/latest/stall_enable.fpga.tar.gz" download>here</a>.

### On a Windows* System

1. Generate the `Makefile` by running `cmake`.
   ```
   mkdir build
   cd build
   ```
   To compile for the Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA, run `cmake` using the command:
    ```
    cmake -G "NMake Makefiles" ..
   ```
   Alternatively, to compile for the Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX), run `cmake` using the command:

   ```
   cmake -G "NMake Makefiles" .. -DFPGA_BOARD=intel_s10sx_pac:pac_s10 -DNO_HYPER_OPTIMIZATION=true
   ```

2. Compile the design through the generated `Makefile`. The following build targets are provided, matching the recommended development flow:

   * Compile for emulation (fast compile time, targets emulated FPGA device):
     ```
     nmake fpga_emu
     ```
   * Generate the optimization report:
     ```
     nmake report
     ```
   * Compile for FPGA hardware (longer compile time, targets FPGA device):
     ```
     nmake fpga
     ```

> **Note**: The Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA and Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX) do not support Windows*. Compiling to FPGA hardware on Windows* requires a third-party or custom Board Support Package (BSP) with Windows* support.

> **Note**: If you encounter any issues with long paths when compiling under Windows*, you may have to create your ‘build’ directory in a shorter path, for example c:\samples\build.  You can then run cmake from that directory, and provide cmake with the full path to your sample directory.

### Troubleshooting
If an error occurs, you can get more details by running `make` with
the `VERBOSE=1` argument:
``make VERBOSE=1``
For more comprehensive troubleshooting, use the Diagnostics Utility for
Intel&reg; oneAPI Toolkits, which provides system checks to find missing
dependencies and permissions errors.
[Learn more](https://www.intel.com/content/www/us/en/develop/documentation/diagnostic-utility-user-guide/top.html).


### In Third-Party Integrated Development Environments (IDEs)

You can compile and run this tutorial in the Eclipse* IDE (in Linux*) and the Visual Studio* IDE (in Windows*). For instructions, refer to the following link: [FPGA Workflows on Third-Party IDEs for Intel&reg; oneAPI Toolkits](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-oneapi-dpcpp-fpga-workflow-on-ide.html).

## Examining the Reports
Locate `report.html` in the `stall_enable_report.prj/reports/` and `stall_free_report.prj/reports/` directories. Open the reports in Chrome*, Firefox*, Edge*, or Internet Explorer*.

On the main report page, scroll down to the section titled `Compile Estimated Kernel Resource Utilization Summary`. Note that the estimated number of **MLAB**s for KernelComputeStallEnable is smaller than that of KernelComputeStallFree. The reduction in MLABs is due to the elimination of the FIFO queue at the end of the cluster.

## Running the Sample

 1. Run the sample on the FPGA emulator (the kernel executes on the CPU):
     ```
     ./stall_enable.fpga_emu     (Linux)
     stall_enable.fpga_emu.exe   (Windows)
     ```
2. Run the sample on the FPGA device:
     ```
     ./stall_enable.fpga         (Linux)
     ./stall_free.fpga           (Linux)
     stall_enable.fpga.exe       (Windows)
     stall_free.fpga.exe         (Windows)
     ```

### Example of Output

```
PASSED: The results are correct
```

### Discussion of Results
There should be no observable difference in results in the example.

Only the area and latency differences are visible in the reports.

## License

Code samples are licensed under the MIT license. See [License.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/License.txt) for details.

Third-party program Licenses can be found here: [third-party-programs.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/third-party-programs.txt).