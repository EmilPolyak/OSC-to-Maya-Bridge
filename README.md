# OSC Bridge to Maya

OSC Bridge is a small Windows utility for routing real-time Open Sound
Control (OSC) data into Autodesk Maya. It was developed to make phones
and other OSC-capable devices practical real-time input controllers for
animation, procedural systems, and live performance.

A phone can provide both interface controls---such as sliders, buttons,
and toggles---and sensor data from its inertial measurement unit (IMU).
OSC Bridge receives that data, maps selected values to named Maya
attributes, optionally scales or reverses them, and forwards them to
Maya through its command port.

The basic data flow is:

**Phone / OSC source → OSC Bridge → Maya `dataReader` node → Maya
attributes / animation**

This makes it possible to drive Maya parameters interactively with a
phone, use phone orientation as a simple motion-capture input, and
record the incoming data as animation.

## Features

-   Receives OSC data from a phone or other OSC-capable source
-   Detects incoming OSC addresses
-   Maps individual OSC values to named Maya attributes
-   Supports multi-value OSC messages through value indices
-   Applies scale factors before values reach Maya
-   Creates corresponding attributes on a Maya `dataReader` node
-   Streams mapped values into Maya in real time
-   Records incoming values as keyframes
-   Supports baking recorded motion onto driven Maya objects
-   Allows mappings to be saved and reused
-   Works with standard Maya workflows including the Connection Editor,
    expressions, and driven relationships

## Requirements

-   Windows
-   Autodesk Maya
-   A phone or other device capable of sending OSC
-   The computer and phone connected to the same local network
-   An OSC controller/sensor application on the phone

The tutorial accompanying this tool uses **Cintian 2**, a free
OSC-capable mobile application, although other applications capable of
transmitting OSC data can also be used.

> **VPN note:** An active VPN that masks or reroutes your computer's
> local network address can prevent the phone and OSC Bridge from
> communicating. If the connection fails, disable the VPN while using
> the bridge.

## Installation

### 1. Install the Maya receiver

Copy the included Maya receiver Python script into your Maya scripts
directory:

``` text
Documents/
└── maya/
    └── <Maya version>/
        └── scripts/
```

Restart Maya after copying the script so Maya reinitializes the scripts
directory.

Open Maya's **Script Editor**, load the receiver script, and execute the
entire script.

A small receiver window should appear.

If Maya crops part of the receiver interface because of display
resolution or DPI scaling, resize the window. The final control in the
interface is **Go to Start Frame**.

### 2. Open Maya's command port

In the Maya receiver window, click:

**Toggle Command Port**

The port status should indicate that it is open. The default Maya
command port used by this tool is:

``` text
12000
```

Maya's command port allows external applications to send commands into
Maya. OSC Bridge uses it to create and update the attributes that carry
the incoming OSC data.

### 3. Start OSC Bridge

Launch the OSC Bridge executable.

Because the executable may be distributed unsigned, Windows may display
a SmartScreen warning. If you obtained the executable from this
repository and intend to run it, use **More info → Run anyway**.

Windows Firewall may also ask whether OSC Bridge should be allowed to
communicate over the network. Network access is required because the
bridge receives OSC packets from the phone and communicates with Maya.

The bridge displays the local IP address that the phone should use.

The OSC input port is:

``` text
9001
```

Port `9001` is fixed in the current version of the bridge.

## Configure the phone

The phone and computer must be connected to the **same Wi-Fi network**.

In your OSC application, configure outgoing OSC with:

``` text
Host / IP: <IP address displayed by OSC Bridge>
Port:      9001
```

Use the IP address shown by OSC Bridge on your own computer. Do not copy
an IP address from a tutorial, screenshot, or another machine.

Once configured, create a simple controller such as a slider and move
it.

OSC Bridge should report the newly received OSC address in its console.
If it does, communication between the phone and bridge is working.

## Create a mapping

Open the mapping window in OSC Bridge and choose **Add Mapping**.

A mapping contains four important pieces of information:

  -----------------------------------------------------------------------
  Setting                             Purpose
  ----------------------------------- -----------------------------------
  OSC Address                         The incoming OSC message to use

  Value Index                         Which value to use when a message
                                      contains one or more values

  Attribute Name                      The Maya attribute that OSC Bridge
                                      will create/update

  Scale Factor                        Multiplier applied to the incoming
                                      value
  -----------------------------------------------------------------------

### Example: slider

A simple slider typically sends one value, so its value index is:

``` text
0
```

For example:

``` text
OSC Address:    <your slider OSC address>
Value Index:    0
Attribute Name: slider
Scale Factor:   10
```

If the original slider sends values from `0` to `1`, a scale factor of
`10` produces values from `0` to `10`.

This can eliminate the need for a separate Multiply/Divide node in Maya.

After adding the mapping, click:

**Create Maya Attributes**

OSC Bridge creates the corresponding attribute on the Maya `dataReader`
node. You can save the mapping when prompted so it can be reused later.

Finally, enable:

**Capture in Maya**

Select `dataReader` in Maya and watch the Channel Box. Moving the phone
control should now update the mapped attribute in real time.

## Connecting incoming data to Maya

The attributes created on `dataReader` can be used anywhere a normal
Maya attribute can be used.

One straightforward method is the **Connection Editor**:

1.  Select the object you want to control.
2.  Open **Windows → General Editors → Connection Editor**.
3.  Load `dataReader` on the source side.
4.  Load the target object on the destination side.
5.  Connect the mapped attribute to the desired Maya attribute.

For example:

``` text
dataReader.slider → pCube1.scaleX
dataReader.slider → pCube1.scaleY
dataReader.slider → pCube1.scaleZ
```

You can also incorporate the incoming values into expressions,
utility-node networks, rigs, procedural systems, cameras, or other
custom setups.

## Using phone orientation

A phone's IMU can provide real-time orientation data that can be mapped
to object rotation in Maya.

In the mobile application used in the tutorial, the relevant values are:

  Orientation     Value Index
  ------------- -------------
  Roll                      8
  Pitch                     9
  Yaw                      10

These indices are specific to the OSC message layout produced by that
application. If you use a different OSC source, inspect its message
structure and select the appropriate indices.

### Radians to degrees

The orientation values used in the tutorial are converted from radians
to degrees with a scale factor of approximately:

``` text
57.3
```

because:

``` text
180 / π ≈ 57.3
```

Example mappings:

``` text
Roll
Value Index:    8
Attribute Name: roll
Scale Factor:   57.3

Pitch
Value Index:    9
Attribute Name: pitch
Scale Factor:   57.3

Yaw
Value Index:    10
Attribute Name: yaw
Scale Factor:   57.3
```

Depending on the phone orientation and Maya axis convention, an axis may
move in the opposite direction from the physical phone. Reverse it by
using a negative scale factor:

``` text
-57.3
```

In the tutorial setup, the phone orientation was matched experimentally
to Maya's X, Y, and Z rotations. Your rig, device orientation, or
coordinate convention may require a different mapping.

## Recording motion in Maya

OSC Bridge can be used as a simple real-time motion-capture input.

Before recording, open Maya's time-slider preferences and use real-time
playback settings appropriate for the scene. The tutorial uses:

``` text
Playback:  Play Every Frame
Max Speed: 24 fps
```

Make sure the timeline is long enough for the intended recording.

Click **Toggle Recording** in the Maya receiver interface. While
recording, Maya advances through the timeline and places keyframes on
the incoming `dataReader` attributes.

Click **Toggle Recording** again to stop.

To review the captured animation without continuing to overwrite it with
live data, disable **Capture in Maya** in OSC Bridge and rewind the
timeline.

The recorded motion now exists as Maya keyframes on `dataReader`.

## Baking the animation to an object

If `dataReader` is driving another object, the recorded motion can be
baked onto that object.

For example:

1.  Select the driven object.
2.  Open **Edit → Keys → Bake Simulation**.
3.  Select the relevant driven channels in the Channel Box.
4.  Configure Bake Simulation to operate on those channels.
5.  Bake the animation.

The resulting animation exists directly on the Maya object and can be
edited in the Graph Editor like any other keyframed animation.

This is useful for phone-driven camera motion, handheld-style movement,
object rotation, performance input, and other forms of lightweight
motion capture.

## Troubleshooting

### The phone sends nothing to OSC Bridge

Check that:

-   the phone and computer are on the same Wi-Fi network;
-   the phone is sending to the IP address currently displayed by OSC
    Bridge;
-   the outgoing OSC port is `9001`;
-   OSC Bridge has permission to communicate through Windows Firewall;
-   an active VPN is not masking or rerouting the local network
    connection.

### OSC Bridge receives data, but Maya does not

Check that:

-   the Maya receiver script has been executed;
-   the Maya command port is open;
-   the command port reports port `12000`;
-   the mapping has been created;
-   **Create Maya Attributes** has been run;
-   **Capture in Maya** is enabled.

### An IMU axis moves in the wrong direction

Change the mapping's scale factor from:

``` text
57.3
```

to:

``` text
-57.3
```

or vice versa.

### Roll, pitch, and yaw do not match Maya X, Y, and Z

Device coordinate systems and Maya's object axes do not necessarily
correspond directly. Test each incoming orientation channel against the
desired Maya rotation channel and connect the combination appropriate
for the setup.

### The Maya receiver window is cropped

Resize the floating window. Maya UI scaling and display DPI settings can
cause part of the interface to be hidden.

## Network and security notes

OSC is designed for lightweight, low-latency communication and is
commonly used for real-time control data. This tool is intended for use
on a local network with devices and OSC sources you trust.

Maya's command port accepts commands from an external process. Only use
the receiver and bridge in a network environment you consider
appropriate, and close or disable the command port when it is not
needed.

## Typical applications

OSC Bridge can be used for:

-   phone-based animation controllers;
-   real-time camera control;
-   handheld camera motion capture;
-   interactive Maya rigs;
-   live performance systems;
-   procedural animation input;
-   physical or sensor-based interfaces;
-   rapid prototyping of OSC-controlled Maya systems.

Because the bridge exposes incoming OSC values as Maya attributes, the
data can become part of a larger Maya dependency graph instead of being
limited to a particular preconfigured interaction.

## Quick-start checklist

``` text
1. Copy the Maya receiver script to the Maya scripts folder.
2. Restart Maya.
3. Execute the receiver script.
4. Toggle the Maya command port ON.
5. Launch OSC Bridge.
6. Put the phone and computer on the same Wi-Fi network.
7. Set the phone's OSC destination to:
      IP   = address displayed by OSC Bridge
      Port = 9001
8. Move a phone control and confirm OSC Bridge receives it.
9. Create a mapping.
10. Create Maya attributes.
11. Enable Capture in Maya.
12. Connect dataReader attributes to Maya parameters.
13. Optionally record and bake the resulting animation.
```

## Tutorial

The repository is accompanied by a tutorial demonstrating the complete
workflow, including a simple slider, IMU roll/pitch/yaw mapping,
real-time Maya control, recording, and baking the captured animation.

------------------------------------------------------------------------

Developed as a practical bridge between OSC-based controllers and
Autodesk Maya for teaching, experimentation, animation, and real-time
performance.
