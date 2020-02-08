---
layout: post
title: Communicating with the Arduino
subtitle: Our foundation for Arduino data communication
tags: [control]
comments: false
---

# Communication Protocol from the Elegoo
With the method we developed to steer the car, our next step was defining a protocol that we could use to communicate with the Elegoo from our computer.


Command   | Function
----------|-----------------------------
`S90`     | Sets the Steering goal to 90º
`s01`     | Sets the delta (change) per loop
`R00`     | Reads the servo position and returns it

This protocol is extremely simple. The ASCII character defines what value to change and the following two bytes define how it should change.



## Implementing the Protocol
Now that we defined a protocol and a method for sending bytes, we can implement its code.

Using the Arduino IDE and the Serial library we initialized the Serial port with a baud-rate of 9600. This is done using the `begin` method of the serial class with the baud-rate as the parameter.


```c
Serial.begin(9600);
```

> To ensure that your program doesn't run until communication is established it would be smart to include a  loop that runs until communication is established. This can be done with the line: `while(!Serial) {;}`

### Parsing Data
Now the Serial library allows you to easily read bytes send to the Elegoo with `Serial.read();` However, we need to make sure that we don't call this when there is nothing to read or if the data is not in the proper format. The `Serial` class comes with a method for checking the current readable bytes t. This lets us wrap the rest of our code in the conditional:

```c
if (Serial.available() > 2) {
  // Rest of code
}
```


#### Bytes
Now, the protocol relies on reviewing the three bytes from the sender. The first byte is converted to ASCII to define what the instruction is. The other two are then merged into an unsigned `int` using basic bit shift operations. Using a right bit shift `>>` the value is divided by 2, the remainder is truncated, and converted to 8-bit binary.

##### Separating the bytes in Python

```python
low = (value >> 8) & 0xFF
high = value & 0xFF
```

##### Merging the bytes in C

```c
byte high = Serial.read();
byte low  = Serial.read();

return high * 256 + low;
```

---
> The only reason for using two bytes is because we originally planned on using the Arduino for both Steering and Motor control. However, the Arduino struggled to handle all the processes we needed from it. and we decided to separate the steering controls from the servo controls. If you are just doing steering controls then you don't need to split the byte.

### Using data
From there we want to check if the character matches one of the commands listed above, and perform the action correlated with the command. Here's the code:

```c
if (Serial.available() > 2) {

    char instruction = Serial.read();
    unsigned int argument = parseBytes();

    switch(instruction) {
      case 'S':
        Steering.setGoal(argument);
        break;

      case 's':
        Steering.setDelta(argument);
        break;

      case 'r':
        Serial.println(Steering.readPhysical());
        break;  
    }
}
```

The above will run whenever there are more than two bytes to be read and it will read the first byte as an ASCII character and the second two as one unsigned integer. Then the program will use the ASCII character to dictate what will be done with the integer.

## Sending Bytes with Python
Now that we have a method of handling instructions for the Elegoo, we need to develop a method for sending the instructions from the main computer. To do this we will once again use Python and the serial library.
>Since we've already described how the Serial library works we won't go too in-depth for this post.

### Pseudo Code
Here is the layout of the class we developed for communicating with the Elegoo.
```python
class Servo:

    def __init__(self, serial_connection):

        self.nano = serial_connection

        time.sleep(4)
        self.set_steering(90)

    def __del__(self):

        # Close Port so no longer connected.
        self.set_steering(90)
        self.nano.close()


    def set_steering(self, value: int):

        byteArray = self.build_packet('S',value)
        self.nano.write(byteArray)


    @staticmethod
    def build_packet(write_type, value) -> bytes:

        packet = bytes([ord(write_type), lowByte, highByte])
        return packet
```

#### Initialization and Destruction
Since we pass a serial connection as a parameter, we can pass the connection as a local variable for access within the class. Then, the program sleeps for 4 seconds as it takes a bit of time for the Elegoo to initialize properly. So the program waits for a second to prevent commands from being lost or buffered incorrectly. Then the program sets the steering to 90 for use as a standard starting point.

We also use a python destructor variable that is called whenever the program ends or the object is deleted to close the serial port and ensure the vehicle is set to a safe default steering angle and velocity.


#### Packaging and Sending Data
The last part of our class works by writing a packet of bytes to the Elegoo that contains the data for setting a servo angle. To simplify this process, the static method `build_packet` returns a byte array based on the parameters set.

>##### Example:
```python
>>> Servo.build_packet("R", 90)
b'R\x00Z'
```
This method works by using the byte shift formulas described previously.

Once we have the data in the right formula we write it to the serial device. From there, our data is sent to the Elegoo where the bytes are interpreted and adjusts the servo accordingly.

#Conclusion
By implementing the above methods, we were able to develop a safe and effective means of sending data to the Elegoo Nano. This data communication is a fundamental aspect of our robot, allowing us to control different aspects, from steering to movement. Using this communication method is simple and allows us to expand it in the future. We use this implementation in our GamePad Controller as well as autonomous control in the future.

> Note: This post was compiled using previous internal documentation.
