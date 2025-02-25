﻿# `Monte Carlo Pi` Sample

Monte Carlo Simulation is a broad category of computation that utilizes
statistical analysis to reach a result. This sample uses the Monte Carlo
Procedure to estimate the value of pi. By inscribing a circle of radius 1
inside a 2x2 square and then sampling a large number of random coordinates
falling uniformly within the square, the value of pi can be estimated using the
ratio of samples that fall inside the circle divided by the total number of
samples.

This method of estimation works for calculating pi because the expected value
of the sample ratio is equal to the ratio of the area of a circle divided by the area of a square: a circle of radius 1 has an area of pi units squared, while a 2x2
square has an area of 4 units squared, yielding a ratio of pi/4. Therefore, to
estimate the value of pi, our solution will be four times the sample ratio.

For comprehensive instructions, see the [Intel&reg; oneAPI Programming Guide](https://software.intel.com/en-us/oneapi-programming-guide) and search based on relevant terms noted in the comments.

| Property            | Description 
|:---                 |:---
| What you will learn | How to start using the SYCL* compliant reduction extension in the compiler
| Time to complete    | 15 minutes


## Purpose

The Monte Carlo procedure for estimating pi is easily parallelized, as each
calculation of a random coordinate point can be considered a discrete work
item. The computations involved with each work item are entirely independent of
one another except for in summing the total number of points inscribed within
the circle. This code sample demonstrates how to utilize the DPC++ reduction
extension for this purpose.

The code attempts to execute on an available GPU and fallback to the
system CPU if a compatible GPU is not detected. The device used for the
compilation is displayed in the output, along with the elapsed time to complete
the computation. A rendered image plot of the computation is also written to a
file.

## Prerequisites

| Optimized for                     | Description
|:---                               |:---
| OS                                | Linux* Ubuntu* 18.04 <br>Windows* 10
| Hardware                          | Skylake with GEN9 or newer
| Software                          | Intel&reg; oneAPI DPC++/C++ Compiler

## Key Implementation Details

The basic SYCL* standard implementation explained in the code includes device selector,
buffer, accessor, kernel, and reduction.

## Building the `Monte Carlo Pi` Program for CPU and GPU

> **Note**: If you have not already done so, set up your CLI
> environment by sourcing  the `setvars` script located in
> the root of your oneAPI installation.
>
> Linux:
> - For system wide installations: `. /opt/intel/oneapi/setvars.sh`
> - For private installations: `. ~/intel/oneapi/setvars.sh`
>
> Windows:
> - `C:\Program Files(x86)\Intel\oneAPI\setvars.bat`
>
>For more information on environment variables, see Use the setvars Script for [Linux or macOS](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-linux-or-macos.html), or [Windows](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-windows.html).


### Running Samples In Intel&reg; DevCloud
If running a sample in the Intel&reg; DevCloud, you must specify the compute node (CPU, GPU, FPGA) and whether to run in batch or interactive mode. For more information, see the Intel&reg; oneAPI Base Toolkit [Get Started Guide](https://devcloud.intel.com/oneapi/get_started/).


### Using Visual Studio Code*  (Optional)

You can use Visual Studio Code (VS Code) extensions to set your environment, create launch configurations,
and browse and download samples.

The basic steps to build and run a sample using VS Code include:
 - Download a sample using the extension **Code Sample Browser for Intel oneAPI&reg; Toolkits**.
 - Configure the oneAPI environment with the extension **Environment Configurator for Intel oneAPI Toolkits**.
 - Open a Terminal in VS Code (**Terminal>New Terminal**).
 - Run the sample in the VS Code terminal using the instructions below.

To learn more about the extensions and how to configure the oneAPI environment, see
[Using Visual Studio Code with Intel® oneAPI Toolkits](https://software.intel.com/content/www/us/en/develop/documentation/using-vs-code-with-intel-oneapi/top.html).

After learning how to use the extensions for Intel oneAPI Toolkits, return to this readme for instructions on how to build and run a sample.

### On Linux*
Perform the following steps:
1. Build the program using the following `cmake` commands.
    ```
    $ mkdir build
    $ cd build
    $ cmake ..
    $ make
    ```

2. Run the program:
    ```
    make run
    ```

3. Clean the program using:
    ```
    make clean
    ```

If an error occurs, you can get more details by running `make` with
the `VERBOSE=1` argument:
```
make VERBOSE=1
```

### Troubleshooting
If you receive an error message, troubleshoot the problem using the Diagnostics Utility for Intel&reg; oneAPI Toolkits, which provides system checks to find missing
dependencies and permissions errors. See [Diagnostics Utility for Intel&reg; oneAPI Toolkits User Guide](https://www.intel.com/content/www/us/en/develop/documentation/diagnostic-utility-user-guide/top.html).

### On Windows* Using Visual Studio* Version 2017 or Newer
- Build the program using VS2017 or VS2019.
    - Right-click on the solution file and open using either VS2017 or VS2019 IDE.
    - Right-click on the project in Solution Explorer and select Rebuild.
    - From the top menu, select Debug -> Start without Debugging.

- Build the program using MSBuild
     - Open "x64 Native Tools Command Prompt for VS2017" or "x64 Native Tools Command Prompt for VS2019"
     - Run the following command: `MSBuild MonteCarloPi.sln /t:Rebuild /p:Configuration="Release"`


## Running the Sample

### Application Parameters

`constexpr int size_n =`

`constexpr int size_wg =`

`constexpr int img_dimensions =`

`constexpr double circle_outline =`

Where:
- `size_n` defines the sample size for the Monte Carlo procedure. 
- `size_wg` defines the size of workgroups inside the kernel code. The number of workgroups is calculated by dividing `size_n` by `size_wg`, so size_n must be greater than or equal to `size_wg`. Increasing `size_n` will increase computation time as well as the accuracy of the pi estimation. Changing `size_wg` will have different performance effects depending on the device used for offloading.
- `img_dimensions` define the size of the output image for data visualization.
- `circle_outline` defines the thickness of the circular border in the output image for data visualization. Setting it to zero will remove it entirely.

### Example of Output
```
Calculating estimated value of pi...

Running on Intel(R) Gen9 HD Graphics NEO
The estimated value of pi (N = 10000) is: 3.15137

Computation complete. The processing time was 0.446072 seconds.
The simulation plot graph has been written to 'MonteCarloPi.bmp'
```
## License

Code samples are licensed under the MIT license. See
[License.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/License.txt) for details.

Third party program Licenses can be found here: [third-party-programs.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/third-party-programs.txt).