# Lab 6 - P "ID" Control
## Task A: Don’t Hit the Wall!!

## Why Proportional?
My TOF sensor is not very accurate when my robot is moving fast.

Furthermore, the slowness of the TOF sensor makes the integral and derivative terms of the PID controller less useful.

When the sensor is slow, the dt value for caculating the integral and derivative terms can make those terms prone to linearization errors, as slow sensors mean large dt values.

These integrator and derivative are most useful when dt is very small.

Additionally, the derivitave term is subject to derivative kick, particularly when I use the e-stop function I built in when something is very close to the front TOF sensor, and the integral term is subject to integrator wind-up, which can cause my car to want to overshoot and consistently hit the wall, and I don't need to break any more TOF sensors.

## Implementing PID Control
To see the global variable declarations, look at the Appendix at the bottom of this page.
### Proportional Control
```
starttime = millis();
dt = (starttime - endtime)*.001;
err[iter] = distance1 - totalDist;
PID = Kp*err[iter];
```
### Integral Control
Wind-up
```
integral = integral + err[iter]*dt;
```

### Derivative Control
LPF?

Source cited reccomends replacing dError with dInput

```
if (iter >2){
    derivative = (err[iter-1] - err[iter])/(dt);
  }
  else {
    derivative = 0;
  }
```
### Putting PID Control Together
```
PID = Kp*err[iter]; + KD * derivative + KI * integral;
```

## Sensor Sampling Rate

## "Hacks" to make the system work & Dealing with Deadband

## Sending Data Over Bluetooth

## Videos of the System Working
### Video 1
### Video 2
### Video 3

## Logged Data & Max Speed Acheived

## Appendix -- Global Variable Declarations
Here is the code I used to declare all the global variables that I used in this lab.
```
int totalDist = 300;
int distance1 = 0;
float Kp = 0.1;
float KI = 0.05;
float KD = 0.05;

int n = 0;
int PID_values[100];
int distance_values[100];
int timestamp_values[100];

int err[1003];
int iter = 0;
int integral = 0;
int derivative = 0;
int endtime = 0;
int starttime;
float dt;
float PID;
```