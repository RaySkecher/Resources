# Input 
## What to send?
Camera/light parameters (position, angle)
Light intensity or color adjustments
Trigger signals (e.g., start frame)

## Cornell box scene geometry is hard coded into the FPGA
We compute and store the resultant frame on the FPGA

## Microbit via UART
Common baud rates like 115200 or 9600 bps.

A baud rate generator (clock divider)

State machine to detect start bit, sample bits at midpoint

Shift registers to assemble/disassemble bytes

Many open-source UART cores available (~100 lines verilog)
https://www.instructables.com/UART-Communication-on-Basys-3-FPGA-Dev-Board-Power/

# Output Options
## VGA Output
Resolution: Stick to 640x480 @ 60Hz
307,200 rays per frame.

### Controller:
Generate horizontal & vertical sync pulses.
Use counters to track pixel (x, y) coordinates.
For each pixel clock cycle, feed coordinates into the raytrace pipeline.
Output the final RGB directly to VGA pins.

#### VGA Spec
Need to send signals to five pins:
|Pin|Usage|
|-|-|
|1|R|
|2|G|
|3|B|
|13|HSync|
|14|VSync|

H Sync tells the monitor to start drawing the next line, like a carriage return
V Sync tells the monitor to move onto the next frame.

A pixel is sent each clock (25 MHz (1 pixel = 40 ns))

Can do `64×48`, upsample with nearest-neighbor or leave black borders.

### Pros/Cons
Looks good, requires no additional hardware to visualize the image

