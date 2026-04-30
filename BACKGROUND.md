# Background

## RGB LCD Panel

We finally get to use the 480×272 RGB LCD panel.

### Interface

```verilog
module lcd
(
    input  rst,
    input  pclk,        

    output LCD_DE,      // Display Enable

    output [4:0] LCD_B, // 5-bit blue color data
    output [5:0] LCD_G, // 6-bit green color data
    output [4:0] LCD_R  // 5-bit red color data
);
```

### Timing Protocol

The RGB Screen follows a very simple protocol. Pixels on the screen are updated in the same sequence as one reads a book... pixels are updated left to right, line by line from top to bottom.

Each horizontal line of pixels is called a scanline. The whole set of scanlines together makes a frame.

<!-- That being said, there are additional "invisible" pixels that are not shown. These are related to synchronizing the frame and scanlines, so we don't "tear" the screen. Older protocols even tried stuffing audio information in these invisible pixels. -->

Notice from the interface definition, that, in addition to the pixel data, the panel requires one more control signal:

- **DE (Data Enable)** — asserted high only during the *active* pixel region; the panel ignores pixel data when DE is low

### Timing Parameters

| Parameter | Horizontal | Vertical |
|-----------|-----------|---------|
| Active region | 480 pixels | 272 lines |
| Buffer Region | 45 clocks | 13 lines |
| **Total per line/frame** | **525 clocks** | **285 lines** |

A scanline has the following region composition:
```
|---ACTIVE---|--BUFFER--|
```

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

## Implementation
You will need to create a module that controls the screen. Each clock cycle writes a pixel to the screen. In order to know which part of the screen you are currently at, you should have 2 variable used to keep track of the x and y position and update them based on the clock. While the screen resultion is 480x272, you will need to take the buffer portion as well for high your x and y variable can go up to(525x285). Then the DE pin is set to high only when in the active region, meaning within the normal 480x272 area. The values on the R, G and B pins, when the clock ticks, is what determines what color will be set for the pixel of that clock cycle.

## Pin Assignments

We are providing the LPF Files that work with the interface provided above. Feel free to adapt that if you choose a different Port.
