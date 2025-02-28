# `Minimum Latency Flow` Sample

This FPGA tutorial demonstrates how to compile your design with the minimum latency flow to achieve low latency at the cost of reduced f<sub>MAX</sub>.

| Area                      | Description
|:---                       |:---
| What you will learn       | How to use the minimum latency flow to compile low-latency designs<br>How to manually override underlying controls set by the minimum latency flow
| Time to complete          | 20 minutes

> **Note**: Even though the Intel DPC++/C++ OneAPI compiler is enough to compile for emulation, generating reports and generating RTL, there are extra software requirements for the simulation flow and FPGA compiles.
>
> For using the simulator flow, Intel® Quartus® Prime Pro Edition and one of the following simulators must be installed and accessible through your PATH:
> - Questa*-Intel® FPGA Edition
> - Questa*-Intel® FPGA Starter Edition
> - ModelSim® SE
>
> When using the hardware compile flow, Intel® Quartus® Prime Pro Edition must be installed and accessible through your PATH.
>
> :warning: Make sure you add the device files associated with the FPGA that you are targeting to your Intel® Quartus® Prime installation.

## Prerequisites

| Optimized for                     | Description
|:---                               |:---
| OS                                | Ubuntu* 18.04/20.04 <br> RHEL*/CentOS* 8 <br> SUSE* 15 <br> Windows* 10
| Hardware                          | Intel® Agilex®, Arria® 10, and Stratix® 10 FPGAs
| Software                          | Intel® oneAPI DPC++/C++ Compiler

This sample is part of the FPGA code samples.
It is categorized as a Tier 3 sample that demonstrates a compiler feature.

```mermaid
flowchart LR
   tier1("Tier 1: Get Started")
   tier2("Tier 2: Explore the Fundamentals")
   tier3("Tier 3: Explore the Advanced Techniques")
   tier4("Tier 4: Explore the Reference Designs")

   tier1 --> tier2 --> tier3 --> tier4

   style tier1 fill:#0071c1,stroke:#0071c1,stroke-width:1px,color:#fff
   style tier2 fill:#0071c1,stroke:#0071c1,stroke-width:1px,color:#fff
   style tier3 fill:#f96,stroke:#333,stroke-width:1px,color:#fff
   style tier4 fill:#0071c1,stroke:#0071c1,stroke-width:1px,color:#fff
```

Find more information about how to navigate this part of the code samples in the [FPGA top-level README.md](/DirectProgramming/C++SYCL_FPGA/README.md).
You can also find more information about [troubleshooting build errors](/DirectProgramming/C++SYCL_FPGA/README.md#troubleshooting), [running the sample on the Intel® DevCloud](/DirectProgramming/C++SYCL_FPGA/README.md#build-and-run-the-samples-on-intel-devcloud-optional), [using Visual Studio Code with the code samples](/DirectProgramming/C++SYCL_FPGA/README.md#use-visual-studio-code-vs-code-optional), [links to selected documentation](/DirectProgramming/C++SYCL_FPGA/README.md#documentation), etc.

## Purpose

This FPGA tutorial demonstrates how to use the minimum latency flow to compile low-latency designs and how to manually override underlying controls set by the minimum latency flow. By default, the minimum latency flow tries to achieve lower latency at the cost of decreased f<sub>MAX</sub>, so this flow is a good starting point for optimizing latency-sensitive designs.

To compile your design with the minimum latency flow, pass the `-Xsoptimize=latency` flag to the `icpx` command.

The minimum latency flow implies the following compiler controls:
- Disable hyper-optimized handshaking on Intel Stratix® 10 and Intel Agilex&trade; devices
- Use zero-latency stall-free clusters exit FIFO
- Disable loop speculation
- Remove the 1-cycle delay on the pipelined loop limiter

The following table shows how users can manually override these underlying controls:
|                                        |Control Flags/Attributes                                    |Reference
|:---                                    |:---                                                        |:---
|Hyper-optimized handshaking             |`-Xshyper-optimized-handshaking=<auto\|off\|on>`            |[Modify the Handshaking Protocol Between Clusters](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-fpga-optimization-guide/top/flags-attr-prag-ext/optimization-flags/hyper-opt-handshaking.html)
|Exit FIFO latency of stall-free clusters|`-Xssfc-exit-fifo-type=<default\|zero-latency\|low-latency>`|[Global Control of Exit FIFO Latency of Stall-free Clusters](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-fpga-optimization-guide/top/flags-attr-prag-ext/optimization-flags/control-exit-fifo-latency.html)
|Loop speculation                        |`[[intel::speculated_iterations(N)]]`                       |[`speculated_iterations` Attribute](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-fpga-optimization-guide/top/flags-attr-prag-ext/loop-directives/speculated-iterations-attribute.html)
|Pipelined loop limiter                  |N/A                                                         |N/A

> **Note**: Using these manual controls overrides the underlying controls individually without affecting other underlying controls introduced by the `-Xsoptimize=latency` compiler flag.

### Understanding the Tutorial Design

The basic function performed by the tutorial kernel is an RGB to grayscale algorithm. To see the impact of the minimum latency flow in this tutorial in terms of latency and f<sub>MAX</sub>, and also see how to override the minimum latency flow with specific manual controls, the design needs to be compiled three times.

Part 1 compiles the design without passing the `-Xsoptimize=latency` flag. In this default flow, the compiler targets higher throughput and f<sub>MAX</sub> with the sacrifice of latency and area.

Part 2 compiles the design with the `-Xsoptimize=latency` flag, so the minimum latency flow is used in this compile. By setting up the underlying compiler controls listed above, the minimum latency flow achieves lower latency by trading off f<sub>MAX</sub>.

Part 3 also compiles the design with the minimum latency flow, as well as manual controls that revert minimum latency flow's default underlying controls. Therefore, latency and f<sub>MAX</sub> of this compile are the same as part 1.

## Key Concepts

* How to use the minimum latency flow to compile low-latency designs
* How to override underlying controls set by the minimum latency flow

## Building the `minimum_latency` Tutorial

> **Note**: When working with the command-line interface (CLI), you should configure the oneAPI toolkits using environment variables. 
> Set up your CLI environment by sourcing the `setvars` script located in the root of your oneAPI installation every time you open a new terminal window. 
> This practice ensures that your compiler, libraries, and tools are ready for development.
>
> Linux*:
> - For system wide installations: `. /opt/intel/oneapi/setvars.sh`
> - For private installations: ` . ~/intel/oneapi/setvars.sh`
> - For non-POSIX shells, like csh, use the following command: `bash -c 'source <install-dir>/setvars.sh ; exec csh'`
>
> Windows*:
> - `C:\Program Files(x86)\Intel\oneAPI\setvars.bat`
> - Windows PowerShell*, use the following command: `cmd.exe "/K" '"C:\Program Files (x86)\Intel\oneAPI\setvars.bat" && powershell'`
>
> For more information on configuring environment variables, see [Use the setvars Script with Linux* or macOS*](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-linux-or-macos.html) or [Use the setvars Script with Windows*](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-windows.html).

### On Linux*

1. Generate the `Makefile` by running `cmake`:
   ```
   mkdir build
   cd build
   ```
   To compile for the default target (the Agilex® device family), run `cmake` using the command:
   ```
   cmake ..
   ```

   > **Note**: You can change the default target by using the command:
   >  ```
   >  cmake .. -DFPGA_DEVICE=<FPGA device family or FPGA part number>
   >  ```
   >
   > Alternatively, you can target an explicit FPGA board variant and BSP by using the following command:
   >  ```
   >  cmake .. -DFPGA_DEVICE=<board-support-package>:<board-variant>
   >  ```
   >
   > You will only be able to run an executable on the FPGA if you specified a BSP.

2. Compile the design using the generated `Makefile`. The following build targets are provided, matching the recommended development flow:

   * Compile for emulation (fast compile time, targets emulated FPGA device):

     ```bash
     make fpga_emu
     ```

   * Generate the optimization reports:

     ```bash
     make report
     ```

   * Compile for simulation (fast compile time, targets simulated FPGA device, reduced data size):

     ```bash
     make fpga_sim
     ```

   * Compile for FPGA hardware (longer compile time, targets FPGA device):

     ```bash
     make fpga
     ```

### On Windows*

1. Generate the `Makefile` by running `cmake`.

   ```
   mkdir build
   cd build
   ```
   To compile for the default target (the Agilex® device family), run `cmake` using the command:
   ```
   cmake -G "NMake Makefiles" ..
   ```
   > **Note**: You can change the default target by using the command:
   >  ```
   >  cmake -G "NMake Makefiles" .. -DFPGA_DEVICE=<FPGA device family or FPGA part number>
   >  ```
   >
   > Alternatively, you can target an explicit FPGA board variant and BSP by using the following command:
   >  ```
   >  cmake -G "NMake Makefiles" .. -DFPGA_DEVICE=<board-support-package>:<board-variant>
   >  ```
   >
   > You will only be able to run an executable on the FPGA if you specified a BSP.

2. Compile the design through the generated `Makefile`. The following build targets are provided, matching the recommended development flow:

   * Compile for emulation (fast compile time, targets emulated FPGA device):
     ```
     nmake fpga_emu
     ```
   * Generate the optimization reports:
     ```
     nmake report
     ```
   * Compile for simulation (fast compile time, targets simulated FPGA device, reduced data  size):
     ```
     nmake fpga_sim
     ``
   * Compile for FPGA hardware (longer compile time, targets FPGA device):
     ```
     nmake fpga
     ```

> **Note**: If you encounter any issues with long paths when compiling under Windows*, you may have to create your ‘build’ directory in a shorter path, for example c:\samples\build.  You can then run cmake from that directory, and provide cmake with the full path to your sample directory.

### Examining the Reports

Locate the pair of `report.html` files in either:

* **Report-only compile**:  `no_control_report.prj`, `minimum_latency_report.prj`, and `manual_revert_report.prj`
* **FPGA hardware compile**: `no_control.fpga.prj`, `minimum_latency.fpga.prj`, and `manual_revert.fpga.prj`

Open the reports in Chrome*, Firefox*, Edge*, or Internet Explorer*.

Navigate to **Loop Analysis** (**Throughput Analysis > Loop Analysis**). In this viewer, you can find the latency of loops in the kernel. The latency of the compile with the minimum latency flow (part 2) should be smaller than the other two compiles. Also, the latency of the other two compiles (part 1 & 3) should be the same.

Navigate to **Clock Frequency Summary** (**Summary > Clock Frequency Summary**) in `no_control.fpga.prj/reports/report.html`, `minimum_latency.fpga.prj/reports/report.html`, and `manual_revert.fpga.prj/reports/report.html` (after `make fpga` completes). In this table, you can find the actual f<sub>MAX</sub>. The f<sub>MAX</sub> of the compile with the minimum latency flow (part 2) should be smaller than the other two compiles. Also, the f<sub>MAX</sub> of the other two compiles (part 1 & 3) should be the same. Note that only the report generated by the FPGA hardware compile will reflect the true f<sub>MAX</sub> affected by the minimum latency flow. The difference is **not** apparent in the reports generated by `make report` because a design's f<sub>MAX</sub> cannot be predicted.

## Running the Sample

1. Run the sample on the FPGA emulator (the kernel executes on the CPU):

   ```bash
   ./no_control.fpga_emu    (Linux)
   no_control.fpga_emu.exe  (Windows)
   ```

2. Run the sample on the FPGA simulator device:

   * On Linux
        ```bash
        CL_CONTEXT_MPSIM_DEVICE_INTELFPGA=1 ./no_control.fpga_sim
        CL_CONTEXT_MPSIM_DEVICE_INTELFPGA=1 ./minimum_latency.fpga_sim
        CL_CONTEXT_MPSIM_DEVICE_INTELFPGA=1 ./manual_revert.fpga_sim
        ```
    * On Windows
        ```bash
        set CL_CONTEXT_MPSIM_DEVICE_INTELFPGA=1
        no_control.fpga_sim.exe
        minimum_latency.fpga_sim.exe
        manual_revert.fpga_sim.exe
        set CL_CONTEXT_MPSIM_DEVICE_INTELFPGA=
        ```

3. Run the sample on the FPGA device (only if you ran `cmake` with `-DFPGA_DEVICE=<board-support-package>:<board-variant>`):

   ```bash
   ./no_control.fpga         (Linux)
   ./minimum_latency.fpga    (Linux)
   ./manual_revert.fpga      (Linux)
   no_control.fpga.exe       (Windows)
   minimum_latency.fpga.exe  (Windows)
   manual_revert.fpga.exe    (Windows)
   ```

## Example Output

Output of sample without minimum latency flow:
```txt
Kernel Throughput: 195.716MB/s
Exec Time: 1.9491e-05s, InputMB: 0.0038147MB
PASSED: all kernel results are correct
```

Output of sample with minimum latency flow:
```txt
Kernel Throughput: 137.764MB/s
Exec Time: 2.769e-05s, InputMB: 0.0038147MB
PASSED: all kernel results are correct
```

Output of sample with minimum latency flow but controls manually reverted:
```txt
Kernel Throughput: 192.934MB/s
Exec Time: 1.9772e-05s, InputMB: 0.0038147MB
PASSED: all kernel results are correct
```

### Discussion of Results

Comparing to Intel Arria® 10 GX FPGA, it is more notable on Intel Stratix® 10 SX FPGA that the minimum latency flow significantly reduces the latency, along with the f<sub>MAX</sub> and the throughput. That is because the minimum latency flow disables the hyper-optimized handshaking, which achieves higher f<sub>MAX</sub> at the cost of increased latency. For more information on the hyper-optimized handshaking protocol on Intel Stratix® 10 and Intel Agilex&trade; devices, see [Modify the Handshaking Protocol Between Clusters](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-fpga-optimization-guide/top/flags-attr-prag-ext/optimization-flags/hyper-opt-handshaking.html).

## License

Code samples are licensed under the MIT license. See [License.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/License.txt) for details.

Third-party program Licenses can be found here: [third-party-programs.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/third-party-programs.txt).
