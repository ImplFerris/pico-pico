## Introduction to Servo Motors

A servo motor controls movement by adjusting its position using a feedback system. It is guided by a signal, usually Pulse Width Modulation (PWM), to reach and maintain the desired position.

They are widely used in applications requiring precise motion, such as robotics, RC vehicles, and camera systems, as well as in various projects.  Hobby servos, which are often used in RC toys like cars, airplanes, are also popular for building robots.

In our exercise, we'll be using the hobby server (Micro Servo SG90)

<img style="display: block; margin: auto;" alt="pico2" src="./images/sg90-servo-motor.jpg"/>

### How does it work?

A servo motor is controlled by sending a series of pulses through its signal line. The signal has a frequency of 50Hz, with a pulse every 20 milliseconds. The width of the pulse determines the servo's position. Typically, a servo can rotate 180 degrees.


### Controlling the position
The position of a servo motor is controlled by sending a pulse with a specific duration. The length of the pulse determines the angle of the motor. For most servos, a 1ms pulse moves the motor to 0 degrees, a 1.5ms pulse moves it to 90 degrees (neutral position), and a 2ms pulse moves it to 180 degrees.

<img style="display: block; margin: auto;" alt="pico2" src="./images/servo-pwm.png"/>

However, from my experiment, I found that not all servos follow these exact timings. For example, with my servo, the pulse duration for 0 degrees was 0.5ms, 1.5ms for 90 degrees, and approximately 2.4ms for 180 degrees. I had to experiment and adjust to get it right. If you're unsure, you can use tools like an oscilloscope to fine-tune it, or simply test different values to find what works best for your specific servo. 

The example I'll provide in this exercise is based on my servo's configuration, you might need to adjust the values depending on the servo you're using.

### Reference
-  [Learn to Control Servo motor using PWM - Wokwi Style](https://blog.wokwi.com/learn-servo-motor-using-wokwi-logic-analyzer/)
 