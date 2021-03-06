Usage
=====

Crazyflie Preparation
---------------------

Since the Crazyflies are sharing radios and communication channels, they need to have a unique identifier/address.
The convention in the Crazyswarm is to use the following address::

    0xE7E7E7E7<X>

where ``<X>`` is the number of the Crazyflie in the hexadecimal system. For example cf1 will use address ``0xE7E7E7E701`` and cf10 uses address ``0xE7E7E7E70A``.
The easiest way to assign addresses is to use the official Crazyflie Python Client.

#. Label your Crazyflies
#. Assign addresses using the Crazyflie Python Client
#. Each radio can control about 15 Crazyflies. If you have more than 15 CFs you will need to assign different channels to the Crazyflies. For example, if you have 49 Crazyflies you'll need three unique channels. It is up to you which channels you assign to which CF, but a good way is to use the Crazyflie number module the number of channels. For example, cf1 is assigned to channel 80, cf2 is assigned to channel 90, cf3 is assigned to channel 100, cf4 is assigned to channel 80 and so on.
#. Upgrade the firmwares of your Crazyflies with the provided firmwares (both NRF51 and STM32 firmwares).
#. Upgrade the firmware of you Crazyradios with the provided firmware.

Your Crazyflie needs to be rebooted after any change of the channel/address for the changes to take any effect.

Adjust Configuration Files
--------------------------

There are two major configuration files. First, we have a config file listing all available (but not necessarily active) CFs::

    # ros_ws/src/crazyswarm/launch/all49.yaml
    crazyflies:
      - id: 1
        channel: 100
        initialPosition: [1.5, 1.5, 0.0]
      - id: 2
        channel: 110
        initialPosition: [1.5, 1.0, 0.0]
      - id: 3
        channel: 120
        initialPosition: [1.5, 0.5, 0.0]

The file assumes that the address of each CF is set as discussed earlier. The channel can be freely configured. The initial position needs to be known for the frame-by-frame tracking as initial guess. It is not required that the CFs start exactly at those positions (a few centimeters variation is fine).

The second configuration file is the ROS launch file (``ros_ws/src/crazyswarm/launch/hover_swarm.launch``). It contains settings on which motion capture system to use and the marker arrangement on the CFs.

Select Motion Capture System
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Below are the relevant settings for the motion capture system::

    # ros_ws/src/crazyswarm/launch/hover_swarm.launch
    # tracking
    motion_capture_type: "vicon" # one of vicon,optitrack
    object_tracking_type: "libobjecttracker" # one of motionCapture,libobjecttracker
    vicon_host_name: "vicon" # only needed if vicon is selected
    optitrack_local_ip: "localhost" # only needed if optitrack is selected
    optitrack_server_ip: "optitrack" # only needed if optitrack is selected

You can choose the motion capture type (currently ``vicon`` or ``optitrack``). The application will connect the the motion capture system using the appropriate SDKs (DataStream SDK and NatNet, respectively). If you select ``libobjecttracker`` as ``object_tracking_type``, the tracking will just use the raw marker cloud from the motion capture system and track the CFs frame-by-frame. If you select ``motionCapture`` as ``object_tracking_type``, the objects as tracked by the motion capture system will be used. In this case you will need unique marker arrangements and your objects need to be named ``cf1``, ``cf2``, ``cf3``, and so on.

Configure Marker Arrangement
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you select the ``libobjecttracker`` as ``motion_capture_type``, you will need to provide the marker arrangement of your markers (identical for all CFs).

#. Place one CF with the desired arrangement at the origin of your motion capture space.
#. Find the coordinates of the used markers
#. Update the config file, see the example below::

    # ros_ws/src/crazyswarm/launch/hover_swarm.launch
    numMarkerConfigurations: 1
    markerConfigurations:
      "0":
        numPoints: 4
        offset: [0.0, -0.01, -0.04] # use this offset if the CF was not placed at the origin
        points:
          "0": [0.0177184,0.0139654,0.0557585]  # coordinates of 1st marker
          "1": [-0.0262914,0.0509139,0.0402475] # coordinates of 2nd marker
          "2": [-0.0328889,-0.02757,0.0390601]  # coordinates of 3rd marker
          "3": [0.0431307,-0.0331216,0.0388839] # coordinates of 4th marker

Monitor Swarm
-------------

A simple GUI is available to enable/disable a subset of the CFs, check the battery voltage, reboot and more.
The tool reads the ``ros_ws/src/crazyswarm/launch/all49.yaml`` file.
You can execute it using::

    ros_ws/src/crazyswarm/scripts
    python chooser.py

An example screenshot is given below:

.. image:: chooser.png

:Clear:   Disables all CFs
:Fill:    Enables all CFs
:battery: Retrieves battery voltage for enabled CFs. Only works if ``crazyflie_server`` is not running at the same time. Can be used while the CF is in power-safe mode.
:version: Retrieves STM32 firmware version of enabled CFs. Only works if ``crazyflie_server`` is not running at the same time. Can only be used if CF is fully powered on.
:sysOff: Puts enabled CFs in power-safe mode (NRF51 powered, but STM32 turned off). Only works if ``crazyflie_server`` is not running at the same time.
:reboot: Reboot enabled CFs (such that NRF51 and STM32 will be powered). Only works if ``crazyflie_server`` is not running at the same time.
:flash (STM): Flashes STM32 firmware to enabled CFs. Only works if ``crazyflie_server`` is not running at the same time. Assumes that firmware is built.
:flash (NRF): Flashes NRF51 firmware to enabled CFs. Only works if ``crazyflie_server`` is not running at the same time. Assumes that firmware is built.


Basic Flight
------------

In order to fly the CFs, the ``crazyflie_server`` needs to be running. Execute it using::

    source ros_ws/devel/setup.bash
    roslaunch crazyswarm hover_swarm.launch

It should only take a few seconds to connect to the CFs. If you have the LED ring extension installed, you can see the connectivity by the color (green=good connectivity; red=bad connectivity). Furthermore, ``rviz`` will show the estimated pose of all CFs. If there is an error (such as a faulty configuration or a turned-off Crazyflie) an error message will be shown and the application exits.

If you have an XBox360 joystick attached to your computer. You can issue a take-off command by pressing "Start" and a landing command by pressing "Back". All CFs should take-off/land in a synchronized fashion, holding the x/y position they were originally placed in.


Advanced Flight
---------------

The flight can be controlled by a python script. A few examples are in ``ros_ws/src/crazyswarm/scripts/``.

#. Test the script in simulation first::

    python figure8_canned.py --sim

(If you are asked to press a button, use the right shoulder on your joystick or press enter on the keyboard.)

#. Run the ``crazyflie_server`` (in another terminal window)::

    source ros_ws/devel/setup.bash
    roslaunch crazyswarm hover_swarm.launch

#. Once the connection is successful, execute the script without ``--sim``::

    python figure8_canned.py
