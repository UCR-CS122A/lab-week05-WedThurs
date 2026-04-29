# Background

## PLL

The iCESugar Pro runs at 25MHz which is way to fast for the RGB LCD. We use an inbuilt primitive found on the FPGA called a Phase Locked Loop or PLL to drop the frequency down from 25MHz to 8MHz.

Configuring PLLs is a process that varies with FPGAs and tooling. While it is possible to figure out the configuration digging through forums and datasheets, OSS-CAD-Suite provides a tool called `ecpppll` specifically for configuring the PLL on the ECP5 FPGA. You can change the frequency with the following tool call:

```bash
ecppll -i 25 -o 8 --highres -f ecppll.v
```

This generates the SV necessary for spitting out . 

## RGB LCD Panel

We finally get to use the 480×272 RGB LCD panel.

### Interface

```verilog
module top
(
    input  rst,
    input  pclk,        // should get 8MHz clk

    output LCD_DE,      // Display Enable
    output LCD_HSYNC,   // Horizontal Sync
    output LCD_VSYNC,   // Vertical Sync

    output [4:0] LCD_B, // 5-bit blue color data
    output [5:0] LCD_G, // 6-bit green color data
    output [4:0] LCD_R  // 5-bit red color data
);
```

### Timing Protocol

To understand the purpose of each of the above wires, we reccomend checking out the following link that illustrates how the [VGA protocol](https://www.cs.ucr.edu/~jtarango/cs122a_lab4.html) works, which the LCD Panel's protocol is loosely based off.

Notive from the interaface definition, that, in addition to the pixel data, the panel requires three more control signals:

- **HSYNC** — horizontal sync pulse, fired once per scan line
- **VSYNC** — vertical sync pulse, fired once per frame
- **DE (Data Enable)** — asserted high only during the *active* pixel region; the panel ignores pixel data when DE is low

It's important to understand, that a full frame consists of more clock cycles than just the visible pixels. The extra cycles are called the **front porch**, **back porch**, and **sync pulse** — they are legacy blanking intervals inherited from the [CRT](https://en.wikipedia.org/wiki/Cathode_ray_tube) (Cathode Ray Tube) era (related to electron-gun timing), but modern LCD panels still require them.

### Timing Parameters

| Parameter | Horizontal | Vertical |
|-----------|-----------|---------|
| Active region | 480 pixels | 272 lines |
| Back porch | 43 clocks | 12 lines |
| Front porch | 2 clocks | 1 line |
| Sync pulse width | 1 clock | 1 line |
| **Total per line/frame** | **526 clocks** | **286 lines** |

There is a diagram on the VGA tutorial. With that, the above terms will make sense.

Given 8MHz, whats the frame rate of your Panel? Is it enough for Valorant?

## RGB Color Encoding — RGB565

Each pixel is represented as a **16-bit value** packed in **RGB565** format. This is similar to the ST7735 encoding scheme for those of you who chose that component for your CS120B Final Project.

```
Bit: 15 14 13 12 11 | 10  9  8  7  6  5 |  4  3  2  1  0
      R4 R3 R2 R1 R0   G5 G4 G3 G2 G1 G0   B4 B3 B2 B1 B0
       5 bits red        6 bits green          5 bits blue
```

| Channel | Bits | Range | Full-intensity value |
|---------|------|-------|----------------------|
| Red | 5 | 0 – 31 | `5'd31` |
| Green | 6 | 0 – 63 | `6'd63` |
| Blue | 5 | 0 – 31 | `5'd31` |

>Green gets one extra bit because the human eye is most sensitive to green.

## Pin Assignments

We are providing the LPF Files that work with the interface provided above. Feel free to adapt that if you choose a different Port.
