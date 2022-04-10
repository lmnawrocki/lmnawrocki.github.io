# Lab 8 - Stunts
## Task A - Flip
Due to a broken TOF sensor, I am currently unable to complete the flipping portion of this lab.

![icri](../images/brokentof.jpg)

However, I have figured out that I absolutely cannot do a flip with open loop control on the tile floor outside of the sticky mat.

### Flip: Theoretically:
In theory, I believe that the following code in a control loop, combined with my Kalman Filter from the previous lab, should lead to my robot flipping and driving back the way it came.
```cpp
if (flipped == false){
      kalmanFilter();
      analogWrite(13, 255); // forward
      analogWrite(6, 255);
  }
  if (abs(mu(0,0)) < 700){ // indicates the robot is very likely on the sticky mat
    flipped = true;
  }
  if (flipped == true) {
    analogWrite(13, 0); // turn forward motors off
    analogWrite(6, 0);
    analogWrite(12, 255); // turn
    analogWrite(7, 255);
  }
```
## Open Loop Stunt

## Blooper Video
[link](https://photos.app.goo.gl/GfRHqqUpcPGycWCg7)