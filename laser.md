<a href="index">
<img src="images/back.png" alt="Back" height="35" width="35">
</a>

## Custom CNC Laser Cutter

---

### Background

This project began as a challenge to myself and a creative outlet, as I was interested in using it for art projects in my spare time, as a stencil cutter. I started out by buying various linear rails, stepper motors and hardware from eBay and Ali Express. Piece by piece, I assembled the CNC machine as time and finances allowed, eventually coming together as a 2-axis CNC laser with end-stops, cooling fan, and a custom built control board.

---
### Final Design

#### Technical Specifications

* 12in x 12in cutting size
* Custom GRBL-based software
* Custom ESP32-based controller board
* 1.8W 445nm M140 Laser Diode with G-2 Glass Lens (30mm)

#### Design

<p align="center">
  <img src="images/laser.JPG" width="100%">
</p>

The design features a 12"x12" cutting plate, which is made from 3/8" clear acrylic. This base plate is suceptable to being cut by the laser, so a 12"x12"x1/8" replaceable plywood base is used on top of the acrylic. The base plate, which is the Y axis of the machine, is driven by a Nema 17 stepper motor and a 350mmx8mm leadscrew with anti-backlash nut. It has a total travel distance of 305mm (~12 in.)

The gantry holds the X axis and laser housing. A Nema 17 stepper motor drives a belt to guide the laser housing module along the 2 linear rails for a total travel distance of 305mm (~12 in.) Both axes have endstops at the origin to home the device and prevent the machine from crashing. Future plans include adding endstops to the far ends of the axes too. 

The laser housing is a block of 1.25"x4" milled aluminum which acts as a heatsink. 2 caps were 3D printed to allow attachment to the housing. The top cap contains an attachment for the cable drag which allows wires to reach the laser diode and fan and holds a small PCB that distributes power and signals to both devices. The bottom cap has a mounting rail to connect the drive belt and adjust the tension. It also has a hole to access the set screw that keeps the laser in place. The laser diode module is contained within a chamber cut down the center of the aluminum block.

#### Controller Board

#### Future Updates

* New custom ESP32 based controller board
* Motor controlled focus (autofocus using ToF sensor?)
* Replace cheap linear rails with more robust V-Slot 
