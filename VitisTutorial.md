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

Good job! You built a component from just C!

## Using your component in Vivado

Open Vivado and create a new RTL project. Select your board in the next step.

<img width="1480" height="919" alt="image" src="https://github.com/user-attachments/assets/3482dbaf-e5da-41f8-9be9-815f619ffb6f" />

Go to `Tools -> Settings`. Then go to `IP -> Repository`. Select the `hls/impl/ip` folder in your Vitis project.

<img width="1895" height="930" alt="image" src="https://github.com/user-attachments/assets/4bf073e0-a0ca-4a5f-ba98-5e252c406096" />

If you now open the IP catalogue (option on the left), you should see your IP block.

<img width="1196" height="445" alt="image" src="https://github.com/user-attachments/assets/b93e074c-cf22-493f-8871-f8ee7545d62a" />

If you see "Repository properties -> Number of IPs: 1" but do NOT see an IP block in the catalogue, you need to set your Vivado project to use the *exact* same board ID as the Vitis project.

Now, we want to be able to use this in our code. To see how to do it, right click the block in the IP catalogue, click "Customize IP", then OK. You should now see this:

<img width="506" height="634" alt="image" src="https://github.com/user-attachments/assets/ec0de1a4-8d61-442e-85f1-5abd91a569e0" />

The important part for us in here is the "Instantiation Template", which will show us how to call it in verilog. Click generate and find the instantiation template under the "IP sources" tab:

<img width="1188" height="562" alt="image" src="https://github.com/user-attachments/assets/d17da815-97f1-430f-b743-48642e30e652" />

Go back to the "Hierarchy" tab and create a new design source called `top.v`. Skip the inputs for now. Paste the instantiation template into your `top.v` file:

<img width="605" height="528" alt="image" src="https://github.com/user-attachments/assets/da61d5ea-311c-445e-9ade-043a4f286ae4" />

Now, you need to give the program inputs. Search up "Basys 3 master constraints file" and copy it into a new constraints source called `basys3.xdc`.

We want to use all of the LEDs and switches, so go into "Column Selection Mode" and remove all of the comments before the LEDs and switches.

<img width="725" height="152" alt="image" src="https://github.com/user-attachments/assets/b1086466-63c3-4739-8bdb-d767410ef712" />

<img width="565" height="570" alt="image" src="https://github.com/user-attachments/assets/5ce09a36-f065-411c-8d6d-ddf5c96a5a05" />

Almost there! We now need to give the loop inputs and outputs so the synthesizer won't optimize it all out.

Update your `top.v` to be something along the lines of:

```v
module top(
    input clk,  // These are all defined in the constraints file
    input[15:0] sw,
    output[15:0] led
    );
    
    wire A_ce0, B_ce0, B_we0, ap_rst, ap_done, ap_idle, ap_ready, ap_start;
    wire[4:0] A_address0, A_q0, B_address0;
    wire[5:0] B_d0;
    
    assign {ap_start, ap_rst, A_q0} = sw[6:0];
    assign {ap_done, ap_ready, ap_idle, A_ce0, B_ce0, A_address0, B_address0} = led[15:0];
    
    loop_perfect_0 your_instance_name (
      .A_ce0(A_ce0),            // output wire A_ce0
      .B_ce0(B_ce0),            // output wire B_ce0
      .B_we0(B_we0),            // output wire B_we0
      .ap_clk(clk),          // input wire ap_clk  --- SET TO CLK
      .ap_rst(ap_rst),          // input wire ap_rst
      .ap_done(ap_done),        // output wire ap_done
      .ap_idle(ap_idle),        // output wire ap_idle
      .ap_ready(ap_ready),      // output wire ap_ready
      .ap_start(ap_start),      // input wire ap_start
      .A_address0(A_address0),  // output wire [4 : 0] A_address0
      .A_q0(A_q0),              // input wire [4 : 0] A_q0
      .B_address0(B_address0),  // output wire [4 : 0] B_address0
      .B_d0(B_d0)              // output wire [5 : 0] B_d0
    );

endmodule
```

Run synthesis and implementation, then open the implemented design. You should be shown a screen like this:

<img width="1487" height="908" alt="image" src="https://github.com/user-attachments/assets/317161e2-c3c2-4d6e-9c53-e7fa09d4a7ed" />

To see the generated schematic, click here:

<img width="409" height="175" alt="image" src="https://github.com/user-attachments/assets/4d37f957-8e2f-41b1-a1e7-3dff28193e5b" />

You can navigate in both of these menus by holding `ctrl` and using mouse operations (drag and scroll).

You can expand logic blocks in the schematic by clicking the `+` icons in the top left corners.

<img width="1164" height="755" alt="image" src="https://github.com/user-attachments/assets/a1b34939-6beb-4914-8190-ca7ec45c01be" />

