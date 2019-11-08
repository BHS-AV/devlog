---
layout: post
title: Introduction and Goals
tags: [intro]
comments: false
---
# Introduction
We are the Brentwood High School Autonomous Vehicles team, a group of four seniors that explore the field of autonomous vehicles.

Our project originally began in August 2018, and over the past year and a half we worked to decode the realm of autonomous vehicles, from understanding the algorithms that power them, to their control schemes. We hope to use our conceptual understanding to create a custom implementation of an autonomous vehicle in a scale car.

# The Car
Our car is built using the [MIT RACECAR](https://mit-racecar.github.io/) platform with some modifications. 

Built on a 1/10th scale Traxxas RC Car, our autonomous vehicle uses various sensors, ranging from a LIDAR, a ZED Camera, and an IMU to obtain environmental information. This information is then locally processed on the embedded Jetson TX1, a development kit built around the NVIDIA Maxwell Architecture, which is optimized for visual and data processing. With the environmental information we control the car using a custom Arduino implementation for steering and a [VESC]() for speed control.

With a custom power management system using a laptop battery, the car is powered and is a self functioning unit.

# First Year
Our first year was mainly exploratory. We dove into the field and learned various different concepts that are integral to autonomous vehicles. These concepts varied from what type of hardware we needed, to our sensor setup, and the basics of algorithmic control.

Our Goals:
- Alter the Traxxas Car
- Build the car chassis
- Mount all the Hardware
- Make a power delivery system
- Flash the Jetson
- Integrate all the hardware with the Jetson

# Second Year