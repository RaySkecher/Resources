## What Vivado does

- Takes in an HDL (ie. verilog)
- Synthesis: convert to logic gates / simplify
- Implementation: determine logic gate placements and wiring
- Simulation: simulate a board to ensure it works as intended
- Bitstream generation: create a "bitstream" which encodes the necessary logic gate info and can be uploaded to the FPGA

## What Vitis does

Converts C/C++/OpenCL code to HDL (ie. verilog), which can then be sent to Vivado.

Also used for "embedded software development" for chips that have FPGAs that have a cpu (like Zynq).

Also provides libraries for tasks like signal processing and AI to make development easier.

## HDL-centered pipeline

- Write design in verilog
- Simulate and verify in Vivado/iverilog/verilator
- Synthesize (Vivado)
- Implement (Vivado)
- Timing analysis and constraints with Vivado. Set clock speed and other constraints to ensure that the design meets timing requirements.
- Bitstream generation & uploading (Vivado)

## HLS pipeline

- Create a base hardware design in Vivado (ie. io, memory controllers, or other peripherals). Include "IP blocks" which are black boxes to be implemented later. Export the hardware design, so Vitis can use it.
- Develop the software using Vitis. Create a project and write your C/C++ code to be run on the FPGA.
- Use the Vitis High Level Synthesis (HLS) tool to compile your code to HDL. This HDL is specific to the hardware design you gave Vitis.
- Build the system (Vitis). Vitis can synthesize, implement, and generate bitstream.
- Debug / verify (Vitis). Vitis provides tools for debugging with HLS (not sure what specifically).
- Upload to FPGA (Vitis).
