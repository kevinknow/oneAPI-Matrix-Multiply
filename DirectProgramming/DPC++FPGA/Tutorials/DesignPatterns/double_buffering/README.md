# Double Buffering to Overlap Kernel Execution with Buffer Transfers and Host Processing
This FPGA tutorial demonstrates how to parallelize host-side processing and buffer transfers between host and device with kernel execution, which can improve overall application performance.

| Optimized for                     | Description
---                                 |---
| OS                                | Linux* Ubuntu* 18.04/20.04 <br> RHEL*/CentOS* 8 <br> SUSE* 15 <br> Windows* 10
| Hardware                          | Intel&reg; Programmable Acceleration Card (PAC) with Intel Arria&reg; 10 GX FPGA <br> Intel&reg; FPGA Programmable Acceleration Card (PAC) D5005 (with Intel Stratix&reg; 10 SX) <br> Intel&reg; FPGA 3rd party / custom platforms with oneAPI support <br> *__Note__: Intel&reg; FPGA PAC hardware is only compatible with Ubuntu 18.04*
| Software                          | Intel&reg; oneAPI DPC++ Compiler <br> Intel&reg; FPGA Add-On for oneAPI Base Toolkit
| What you will learn               | How and when to implement the double buffering optimization technique
| Time to complete                  | 30 minutes

## Purpose
In an application where the FPGA kernel is executed multiple times, the host must perform the following processing and buffer transfers before each kernel invocation.
1. The output data from the *previous* invocation must be transferred from the device to the host and then processed by the host. Examples of this processing include:
   * Copying the data to another location
   * Rearranging the data
   * Verifying it in some way
2. The input data for the *next* invocation must be processed by the host and then transferred to the device. Examples of this processing include:
   * Copying the data from another location
   * Rearranging the data for kernel consumption
   * Generating the data in some way

Without double buffering, host processing and buffer transfers occur *between* kernel executions. Therefore, there is a gap in time between kernel executions, which you can refer to as kernel *downtime* (see diagram below). If these operations overlap with kernel execution, the kernels can execute back-to-back with minimal downtime, thereby increasing overall application performance.

### Determining When is Double Buffering Possible

Let's define the required variables:
- **R** = Time to transfer the kernel's output buffer from device to host.
- **Op** = Host-side processing time of kernel output data (*output processing*)
- **Ip** = Host-side processing time for kernel input data (*input processing*)
- **W** = Time to transfer the kernel's input buffer from host to device.
- **K** = Kernel execution time

![](downtime.png)

In general, **R**, **Op**, **Ip**, and **W** operations must all complete before the next kernel is launched. To maximize performance, while one kernel is executing on the device, these operations should execute simultaneously on the host and operate on a second set of buffer locations. They should complete before the current kernel completes, allowing the next kernel to be launched immediately with no downtime. In general, to maximize performance, the host must launch a new kernel every **K**.

This leads to the following constraint:

```c++
R + Op + Ip + W <= K, in order to minimize kernel downtime.
```
If the above constraint is not satisfied, a performance improvement may still be observed because *some* overlap (perhaps not complete overlap) is still possible. Further improvement is possible by extending the double buffering concept to N-way buffering (see the corresponding tutorial).

### Measuring the Impact of Double Buffering

You must get a sense of the kernel downtime to identify the degree to which this technique can help improve performance.

This can be done by querying the total kernel execution time from the runtime and comparing it to the overall application execution time. In an application where kernels execute with minimal downtime, these two numbers will be close. However, if kernels have a lot of downtime, the overall execution time will notably exceed kernel execution time. The tutorial code exemplifies how to do this.

### Tutorial Implementation Notes

The basic idea is to:
1. Perform the input processing for the first two kernel executions and queue them both.
2. Immediately call the `process_output()` method (automatically blocked by the SYCL* runtime) on the first kernel completing because of the implicit data dependency.
3. When the first kernel completes, the second kernel begins executing immediately because it was already queued.
4. While the second kernel runs, the host processes the output data from the first kernel and prepares the third kernel's input data.
5. As long as the above operations complete before the second kernel completes, the third kernel is queued early enough to allow it to be launched immediately after the second kernel.

The process then repeats.

The impact of double buffering on the total runtime of the tutorial program will be analyzed in the "Running the Sample" section below.

### Additional Documentation
- [Explore SYCL* Through Intel&reg; FPGA Code Samples](https://software.intel.com/content/www/us/en/develop/articles/explore-dpcpp-through-intel-fpga-code-samples.html) helps you to navigate the samples and build your knowledge of FPGAs and SYCL.
- [FPGA Optimization Guide for Intel&reg; oneAPI Toolkits](https://software.intel.com/content/www/us/en/develop/documentation/oneapi-fpga-optimization-guide) helps you understand how to target FPGAs using SYCL and Intel&reg; oneAPI Toolkits.
- [Intel&reg; oneAPI Programming Guide](https://software.intel.com/en-us/oneapi-programming-guide) helps you understand target-independent, SYCL-compliant programming using Intel&reg; oneAPI Toolkits.

## Key Concepts
* The double buffering optimization technique
* Determining when double buffering is beneficial
* How to measure the impact of double buffering


## Building the `double_buffering` Tutorial

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

If running a sample in the Intel&reg; DevCloud, remember that you must specify the
type of compute node and whether to run in batch or interactive mode. Compiles
to FPGA are only supported on fpga_compile nodes. Executing programs on FPGA
hardware is only supported on fpga_runtime nodes of the appropriate type, such
as fpga_runtime:arria10 or fpga_runtime:stratix10. Neither compiling nor
executing programs on FPGA hardware are supported on the login nodes. For more
information, see the Intel&reg; oneAPI Base Toolkit Get Started Guide
([https://devcloud.intel.com/oneapi/documentation/base-toolkit/](https://devcloud.intel.com/oneapi/documentation/base-toolkit/)).

When compiling for FPGA hardware, it is recommended to increase the job timeout
to 12h.


### Using Visual Studio Code*  (Optional)

You can use Visual Studio Code (VS Code) extensions to set your environment,
create launch configurations, and browse and download samples.

The basic steps to build and run a sample using VS Code include:
 - Download a sample using the extension **Code Sample Browser for Intel&reg; oneAPI Toolkits**.
 - Configure the oneAPI environment with the extension **Environment Configurator for Intel&reg; oneAPI Toolkits**.
 - Open a Terminal in VS Code (**Terminal>New Terminal**).
 - Run the sample in the VS Code terminal using the instructions below.

To learn more about the extensions and how to configure the oneAPI environment, see
[Using Visual Studio Code with Intel&reg; oneAPI Toolkits User Guide](https://software.intel.com/content/www/us/en/develop/documentation/using-vs-code-with-intel-oneapi/top.html).

After learning how to use the extensions for Intel oneAPI Toolkits, return to
this readme for instructions on how to build and run a sample.

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
   Alternatively, to compile for the Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX), run `cmake` using the command:

   ```
   cmake .. -DFPGA_BOARD=intel_s10sx_pac:pac_s10
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
   * Generate the optimization report:
     ```
     make report
     ```
   * Compile for FPGA hardware (longer compile time, targets FPGA device):
     ```
     make fpga
     ```
3. (Optional) As the above hardware compile may take several hours to complete, FPGA precompiled binaries (compatible with Linux* Ubuntu* 18.04) can be downloaded <a href="https://iotdk.intel.com/fpga-precompiled-binaries/latest/double_buffering.fpga.tar.gz" download>here</a>.

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
   cmake -G "NMake Makefiles" .. -DFPGA_BOARD=intel_s10sx_pac:pac_s10
   ```
   You can also compile for a custom FPGA platform. Ensure that the board support package is installed on your system. Then run `cmake` using the command:
   ```
   cmake -G "NMake Makefiles" .. -DFPGA_BOARD=<board-support-package>:<board-variant>
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

*Note:* The Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA and Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX) do not support Windows*. Compiling to FPGA hardware on Windows* requires a third-party or custom Board Support Package (BSP) with Windows* support.<br>
*Note:* If you encounter any issues with long paths when compiling under Windows*, you may have to create your ‘build’ directory in a shorter path, for example c:\samples\build.  You can then run cmake from that directory, and provide cmake with the full path to your sample directory.

 ### Troubleshooting
If an error occurs, you can get more details by running `make` with
the `VERBOSE=1` argument:
``make VERBOSE=1``
For more comprehensive troubleshooting, use the Diagnostics Utility for
Intel&reg; oneAPI Toolkits, which provides system checks to find missing
dependencies and permissions errors.
[Learn more](https://software.intel.com/content/www/us/en/develop/documentation/diagnostic-utility-user-guide/top.html).

 ### In Third-Party Integrated Development Environments (IDEs)

You can compile and run this tutorial in the Eclipse* IDE (in Linux*) and the Visual Studio* IDE (in Windows*). For instructions, refer to the following link: [FPGA Workflows on Third-Party IDEs for Intel&reg; oneAPI Toolkits](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-oneapi-dpcpp-fpga-workflow-on-ide.html)

## Examining the Reports
Locate `report.html` in the `double_buffering_report.prj/reports/` directory. Open the report in any of Chrome*, Firefox*, Edge*, or Internet Explorer*.

Note that because the optimization described in this tutorial occurs at the *runtime* level, the FPGA compiler report will not show a difference between the optimized and unoptimized cases.


## Running the Sample

 1. Run the sample on the FPGA emulator (the kernel executes on the CPU):
     ```
     ./double_buffering.fpga_emu     (Linux)
     double_buffering.fpga_emu.exe   (Windows)
     ```
2. Run the sample on the FPGA device:
     ```
     ./double_buffering.fpga         (Linux)
     double_buffering.fpga.exe       (Windows)
     ```

### Example of Output

```
Platform name: Intel(R) FPGA SDK for OpenCL(TM)
Device name: pac_a10 : Intel PAC Platform (pac_ee00000)


Executing kernel 100 times in each round.

*** Beginning execution, without double buffering
Launching kernel #0
Launching kernel #10
Launching kernel #20
Launching kernel #30
Launching kernel #40
Launching kernel #50
Launching kernel #60
Launching kernel #70
Launching kernel #80
Launching kernel #90

Overall execution time without double buffering = 29742 ms
Total kernel-only execution time without double buffering = 17856 ms
Throughput = 35.255249 MB/s


*** Beginning execution, with double buffering.
Launching kernel #0
Launching kernel #10
Launching kernel #20
Launching kernel #30
Launching kernel #40
Launching kernel #50
Launching kernel #60
Launching kernel #70
Launching kernel #80
Launching kernel #90

Overall execution time with double buffering = 17967 ms
Total kernel-only execution time with double buffering = 17869 ms
Throughput = 58.35976 MB/s


Verification PASSED
```

### Discussion of Results

A test compile of this tutorial design achieved a maximum frequency (f<sub>MAX</sub>) of approximately 340 MHz on the Intel&reg; Programmable Acceleration Card with Intel&reg; Arria&reg; 10 GX FPGA. The results with and without double buffering are shown in the following table:

Configuration | Overall Execution Time (ms) | Total Kernel Execution time (ms)
|:-- |:-- |:--
Without double buffering | 23462 | 15187
With double buffering | 15145 | 15034

In both runs, the total kernel execution time is similar, as expected. However, without double buffering, the overall execution time notably exceeds the total kernel execution time, implying there is downtime between kernel executions. With double buffering, the overall execution time is close to the total kernel execution time.

## License
Code samples are licensed under the MIT license. See
[License.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/License.txt) for details.

Third party program Licenses can be found here: [third-party-programs.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/third-party-programs.txt).