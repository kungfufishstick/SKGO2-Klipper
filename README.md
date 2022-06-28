# SKGO2-Klipper

Klipper config for custom SKGO2

This file is compiled from the Klipper github for Big Tree Tech SKR v1.3 and configured forthe  SecKit Go 2.

## Printer Specifications
- Core XY SecKit Go 2
- BTT SKR v1.3
- BL Touch v3.1
- TMC2209 stepper driver motors
- XYZ Linear Rails
- X Y Z Axis Moons Stepper NEMA 17 1.8 degree
- Extruder - Pancake NEMA 17 1.8
- Single 40x40x20 blower motor for part cooling
- Cooling duct with volcano nozzle:
    - [Thingiverse Volcano Nozzle Cooling Duct](https://www.thingiverse.com/thing:4594501)
- Bondtech BMG Extruder
- E3D Volcano Hotend
- 12864 LCD

## Axis Details
- X/Y motors attached to 16T gear and moves 2mm pitch belt
- Z motor connects to 16T gear and moves 2mm pitch belt that runs to dual Z lead screws with 20T gear. The lead screw is a T8, so it is 2mm pitch and 4 threads.
- Build Volume: 310x310x350 mm^3

| :warning: NOTICE |
|------------------|
Microsteps for this build is actually 32. 

## Raspberry Pi Details

- Uses Octopi with KIAUH. 
- Currently runs Moonraker with fluidd.


## Helpful Guides

**<details><summary>Printer wont come back up for Octoprint following restart.</summary>**

</br>

If the printer has a blank screen after a restart, and Octoprint can't connect at all, I have needed to force restart using Putty. 

After logging in, the following commands should do the trick:

```
sudo service klipper stop
sudo service klipper start
```
</details>





**<details><summary>Calibrating Rotation Distance for X, Y, Z</summary>**

</br>

| Information found at: |
| --- |
| [Klipper3d Rotation Distance](https://www.klipper3d.org/Rotation_Distance.html) |
| [Hardware Inspection for Rotation Distance](https://www.klipper3d.org/Rotation_Distance.html#obtaining-rotation_distance-by-inspecting-the-hardware) |

The X, Y, and Z axis are all the same motor, fixed to a 16T pulley. Z is different since it runs to dual Z axis T8 lead screws that are each fixed to 20T pulleys, but the rotation distance will hopefully be the same since the motor runs with a 16T pulley. 

Since I wiped my Pi, I am going to start off using the rotation distance by inspecting my hardware. 

All Axis are run with a 16T pulley attached to a Moons Stepper NEMA 17 1.8 degree motor. 

My formula for each axis should be as follows: 

```
rotation_distance = <belt_pitch> * <number_of_teeth_on_pulley>
```

All belts are 2mm pitch and each pulley has 16T.

```
rotation_distance = 2 * 16

rotation_distance = 32
```

| :warning: NOTE |
|--------------------------------------------------------|
| ~~I will be testing the Z axis to make sure. It looks correct on paper but feels off. I'm pretty sure I will need to follow the guide for lead screws.~~  |

| :warning: UPDATE |
|------------------|
| This turned out to be true. Microsteps were also wrong and needed to be set to 32. So microsteps 32 and rotation distance 8.

If the lead screw guide turns out to be the correct one then the formula and result is as follows:

```
rotation_distance = <screw_pitch> * <number_of_separate_threads>

rotation_distance = 2 * 4

rotation_distance = 8
```

<br>

*<details><summary>Using Previously Calculated Step Distance</summary>*

</br>

**Find out stepper motor type.** 

- This build uses Moons Stepper NEMA 17 1.8 degree, which gives means full_steps_per_rotation = 200

**Figure out microsteps.** 

- This build uses TMC2209 and configured for 16 microsteps

**Figure out step distance.** 

- Previous measurements/adjustments could be used but since wiping my Raspberry Pi, I decided to start from the beginning. I will be setting it up to use the default suggested values found on the rotation distance hardware inspection page of Klipper3d.

If I were to use the step distance previously calculated, I would come up with the following:

```
rotation_distance = <full_steps_per_rotation> * <microsteps> * <step_distance>
```

*Round to nearest whole number if within .01.*

```
X Axis Rotation Distance

rotation_distance = 200 * 16 * .00503 
rotation_distance = 16.096 (rounded to 16)
```

```
Y Axis Rotation Distance

rotation_distance = 200 * 16 * .00500 
rotation_distance = 16
```

```
Z Axis Rotation Distance

rotation_distance = 200 * 16 * .00125 
rotation_distance = 4
```

</details>

</details>





**<details><summary>Calibrating Rotation Distance For Extruder</summary>**

| Information found at: |
| --- |
| [Klipper3d Rotation Distance](https://www.klipper3d.org/Rotation_Distance.html) |

Use Measure and Trim method to calibrate rotation_distance for the Extruder. Can use previous measurements, if you have them. I'm starting from scratch.

<details><summary>Using previously calculated step distance</summary>

</br>

***Formula:***
```
rotation_distance = <full_steps_per_rotation> * <microsteps> * <step_distance>
```

*Round to nearest whole number if within .01.*

```
Extruder rotation_distance = 200 * 16 * .001193 (3.8176 round up 4)
```

1. Make sure the extruder has filament in it, the hotend is heated to an 
 	appropriate temperature, and the printer is ready to extrude.
2. Use a marker to place a mark on the filament around 70mm from the intake 
of the extruder body. Then use a digital calipers to measure the actual 
	distance of that mark as precisely as one can. Note this as 
	<initial_mark_distance>.
3. Home all axis to get in "printer ready" state
4. Lift up your nozzle by 50mm (to make room for the filament!)
5. Execute the following commands (one by one)
  a) G92 E0 -This resets the "extruded material" value to 0.
  b) G1 E50 F60
6. This extrudes 50mm filament with 60mm/min.
7. It is important to use the slow extrusion rate for this test as a faster 
	rate can cause high pressure in the extruder which will skew the results. 
	(Do not use the "extrude button" on graphical front-ends for this test 
	as they extrude at a fast rate.)
7. Use the digital calipers to measure the new distance between the extruder 
	body and the mark on the filament. Note this as <subsequent_mark_distance>. 
	Then calculate: 
	actual_extrude_distance = <initial_mark_distance> - <subsequent_mark_distance>
</details>
</details>





**<details><summary>Calibrate Bed Screws</summary>**

First run BED_SCREWS_ADJUST through the terminal tab in Octoprint

Perform the paper test at each point.

Type ADJUSTED into the terminal if you adjusted the bed screw by 1/8th of a turn
or more.
When satisfied type ACCEPT into the terminal to move on.

ACCEPT/ADJUSTED across each screw point.

After doing this, G28 and type SCREWS_TILT_CALCULATE into the terminal to use the 
BL Touch to probe the bed and klipper will return values for bed screw rotation.

Rinse and repeat G28 and SCREWS_TILT_CALCULATE until you are satisfied with the
bed leveling.

</details>




**<details><summary>Other Info</summary>**

Explore docs on the klipper github. Some things of note for this setup are:

- BLTouch.md
- Bed_Level.md
- Bed_Mesh.md
- Pressure_Advance.md
- Probe_Calibrate.md
- Resonance_Compensation.md
- Sensorless_Homing.md
- config_checks.md 
- Verification checks and PID calibration

</details>
		
| :warning: EMPHASIS ON BED MESH |
| --- |
| The way bed mesh handles positioning has changed. It is now based on the probe position, not the nozzle. Min and max positions need to account for probe offset. |

<br>

### Older Guides

<details><summary>Adjusting E-Steps (Step Distance) by user ikarisan</summary>

|Information found at: |
| --- |
| https://github.com/KevinOConnor/klipper/issues/934 |

1. Mark you filament 120mm above the entry to your extruder.
2. Heat up the nozzle to your desired printing temperature
3. Home all axis to get in "printer ready" state
4. Lift up your nozzle by 50mm (to make room for the filament!)
5. Execute the following commands (one by one)
a) G92 E0 -This resets the "extruded material" value to 0.
b) G1 E100 F100
6. This extrudes 100mm filament with 100mm/min.
7. Now measure the distance between your extruder entry and the mark on 
your filament.

Example: If it is 28mm instead of 20mm (120mm - 100mm) then you are 
UNDERextruding by 8mm ==> 92mm instead of 100mm. If it shows 15mm 
then your are OVERextruding by 5mm ==> 105mm.

Now calculate:

c := current value in your config
m := measurement of left over filament
d := desired mm
n := new value in your config

**Formula:**
```
((120 - m) / d) * c = n
```

**Current and adjusted result**
```
((120 - 28) / 100) * 0.010500 = 0.009660
```
| :exclamation: Current step_distance |
|:---:|
| **0.001193** |

Play around to fine tune.

</details>


<details><summary>Calibrating steps for X, Y, and Z Axis</summary>

Print the cube found at:
https://www.thingiverse.com/thing:1278865

You can also just print any 20x20 cube, like CHEPs calibration cube.
Use your digital caliper for measurements.

The instructions there are pretty good but it is for steps and not step distance.
To find steps based on the step_distance in your config, you need to divide
your step_distance by 1. 

step_distance on the x stepper for this config is .00503
1/.00503 rounds up to 198.81 steps. 

When you calibrate steps based on the 20x20 cube, you can take your result and
divide it by one to put it back into step_distance measurement
1/198.81 rounds up to .00503 

The formula is:
e = expected dimension
o = observed dimension
s = current number of steps per mm

(e/o) * s = adjusted steps
divide adjusted steps by 1 to get your step_distance

update your config and reprint the 20x20 cube until satisfied.

</details>

<br>
		
See the example.cfg file for a description of available parameters.

Resonance Compensation has not yet been setup.
