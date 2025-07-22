# Vitis tutorial

## Install

Download the self-extracting web installer for your operating system (likely Windows) from the [Xilinx downloads page](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vitis/2024-2.html).

Run it, and select Vitis:

<img width="870" height="611" alt="image" src="https://github.com/user-attachments/assets/2087742f-7cf6-4403-85e4-405ec0dc26f4" />

Deselect devices that you won't use (you can reinstall them later, if needed):

<img width="740" height="551" alt="image" src="https://github.com/user-attachments/assets/33491bc9-e333-4bfe-b3be-38209770074d" />

## Create example project

Select `Examples`.

<img width="955" height="566" alt="image" src="https://github.com/user-attachments/assets/585813a6-c19b-4621-b20d-4d9a5adbba35" />

Collapse the random examples at the top because we want the "HLS examples". Select the `perfect_loop` example (under `HLS Examples/Vitis HLS Introductory Examples/Pipelining/Loops`).

<img width="323" height="469" alt="image" src="https://github.com/user-attachments/assets/02a9e6bc-ff7e-4b10-952a-fc287283b326" />

Select "create HLS component from template", assign it a location, and create.

<img width="1410" height="909" alt="image" src="https://github.com/user-attachments/assets/d46b49ef-d908-4c82-acbc-50232455cc72" />

Your screen should look like this.

<img width="2510" height="1372" alt="image" src="https://github.com/user-attachments/assets/084e0e8b-ccde-42d3-aa98-5bf7cf23472e" />

## Navigating Vitis

Starting in the top left, your "components" are the generated elements which you will use C/C++ to generate. Under the component name,

- Settings provide configuration for your component (such as which device it's targeting)
- Includes provides libraries for your code to use
- Sources contains the source files for your component (ie. `.c`, `.cpp`)
- Test bench contains unit test -like tests to isolate and verify behavior in your code
- Output will contain generated files

In the bottom left (the "flow"),

- C simulation compiles your code and runs the test benches
- C synthesis converts your C/C++ code to logic gates
- C/RTL cosimulation tests your C code's behavior against the generated logic gates to ensure equivalent behavior
- Package groups generated outputs for heterogenous chips. **NOT** needed for Artix-7.
- Implementation places and routes the logic gates on the FPGA (similar to placement in OpenLane)

Open `sources -> loop_perfect.cpp`.

On the right bar,

- Outline represents the code structure of the currently opened file
- Memory inspector lets you debug the memory of your target hardware during a live debug session
- HLS directives shows information given to the HLS toolchain by your code, which will be important for pipelining and optimization.

## Using Vitis

Try running `C simulation -> run`. If prompted for some feature, select whatever (it doesn't matter now). If you deselected some devices, you may see an error like this:

<img width="1621" height="225" alt="image" src="https://github.com/user-attachments/assets/1323a30a-48e7-420e-aef5-533fe7776f52" />

Read the error and go to settings to solve the issue. You may want to search up the chip being used in your FPGA. Once fixed, run it again.

After that, run C synthesis.

Then, run implementation. This may take a while (took ~2min on my laptop).

Now, look under `syn/verilog` to see the generated verilog.

Good job! You built a component from just C! I will work on updating this later tonight or tomorrow on how to open it in Vivado, so stay tuned.
