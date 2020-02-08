---
layout: post
title: Steering Control
subtitle: Controlling our steering servo with an Arduino
gh-repo: bhs-av/software
tags: [control]
comments: false
---

While working on controlling the VESC, we also began figuring out how we wanted to steer the vehicle.
The steering servo could only be controlled using PWM signals, so we needed to find a PWM capable device.
We researched many ways we could do this until we settled on using the Elegoo Nano, a cheap microcontroller based on the Arduino Nano platform.


## Elegoo Nano
Because we used the Elegoo Nano we had to change the settings of the Arduino IDE so that we could program the vehicle. To do this:

- Set the board to `Arduino Nano`
- Change the processor to the `Legacy Boot Loader`
- Change the programmer so that the Arduino is run as `ISP`

Following these steps should prevent your Elegoo Nano frying itself.

> If you are just using a Arduino Nano, the above does not apply.

## Communication

The easiest solution to sending signals to the vehicle was to use the Arduino Servo Library.

The Arduino IDE comes with an example program you can run to test the library. Using the table below, wire the board to the vehicle

Physical Wiring | Which Port to Attach to
----------------|------------------------
Red Wire        | 5.5V
Black Wire      | GND
White Wire      | D2-D12<sup>1</sup>

> <sup>1</sup> Or any PWM capable port on your board.

The line `myservo.attatch(9)` in the example program describes the port where your board will check for data. You can change this to any PWM capable port on your board.

The sweep program should be heavily documented, explaining every step taken by the program.

## Smooth Motions
The example program slowly steers the wheels from left to right and then back to their starting position. By incrementing by 1 this makes the process very slow. However, if it were to make a large change, it would be very unhealthy for the servo and could damage the components.

Even though it is unlikely that we will try to make such a big jump in servo angles angles, we wanted to implement a safety net if any errors were to occur in the future.

### Delta and Goal

Our implementation involves setting a goal, and moving the physical position of the servo a little closer to the goal each run. Because the loop is running multiple times a second the goal is reached nearly instantly. This makes the angle changes smooth and less likely to damage the vehicle. Instead of instantly changing its angles, the servo now follows a curve, approaching the goal slowly.

To do this we create the variables `delta` and `goal` to go along with the real value.

`delta` is the amount that the real value is incremented per loop.

`goal` is the position we want to set the servo to.

### Error and Sign

We also create the variables `error` and `sign` for incremental calculations

`error` is the difference between the real value and the goal

`sign` is either a negative one or a positive one and is used to differentiate from incrementing up or down, or in terms of steering, left or right.


### Implementation
Here is the final implementation of our smoothing algorithm.

```c
int error = goal - value;
int sign = abs(error) / error;

if (error == 0 ) { return;}

if (abs(error) > delta)
  value += delta*sign;
else
  value = goal;    
```

## Abstraction
Since our main loop is already going to be cluttered handling serial communications from the Jetson. We created a class to handle all of our communication between the servo and the Arduino.

We began this by defining how we wanted the class to work, simplifying it into the following:

- Connect to the vehicle using a specified pin behind the scenes so a future developer does not have to know how it works
- Implement the smoothing algorithm with a method that would be called every cycle
- Methods for manipulating the `goal` and `delta` variables without causing sudden shifts
- Methods for directly reading the physical position of the servo

The resulting class looks similar to this:

```cpp
Traxxas::Traxxas(int Pin, int Throttle, int Delta, int Value, int Goal) {

  Servo serv;
  int pin;
  int throttle;
  int delta;
  int value;
  int goal;
}

Traxxas::~Traxxas() {

}

void Traxxas::refresh() {
    // Smoothing Algorithm
    serv.write(value);
}

void Traxxas::startup() {
    // Attatching the arduino to a pin.
    serv.attach(pin);
}



void Traxxas::setGoal( unsigned int argument) {
  // Sets the goal to the argument
  goal = argument;
}

void Traxxas::setDelta( unsigned int value) {
  // Manipulate the Delta Variable
  delta = value;
}

unsigned int Traxxas::readPhysical() {

  return serv.readMicroseconds();
}
```

> Note: A `Traxxas` object should be initialized as a global object in order to access the start methods in the startup method, and to update the servo from the main loop.

So now that we have this class, we can safely manipulate our servo from the main method.

# Conclusion
With the above implementation, we are able to control the steering mechanism of the Traxxas car with the Elegoo Nano. Furthermore, we are able to prevent servo degradation through our smoothing algorithm. With the foundations for vehicle control, steering, we are able to work toward automation, creating navigation algorithms and processing all input data after we integrate steering control in the next post using the VESC.

> Note: This post was compiled using previous internal documentation.
