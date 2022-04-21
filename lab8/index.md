# Lab 8 - Stunts

In this lab, I will attempt to do some stunts with my robot: getting it to flip near a wall and doing some open loop shenanigans because I can.

## Task A - Flip
Due to a broken TOF sensor, I am currently unable to complete the flipping portion of this lab.

![icri](../images/brokentof.jpg)

However, I have figured out that I absolutely cannot do a flip with open loop control on the tile floor outside of the sticky mat.

### Flip: Theoretically:
In theory, I believe that the following code in a control loop, combined with my Kalman Filter from the previous lab, should lead to my robot flipping and driving back the way it came.
```cpp
  if (flipped == false){
      kalmanFilter();
      analogWrite(13, 255); // move forward until the robot is close to the wall
      analogWrite(6, 255);
  }
  if (abs(mu(0,0)) < 250 && abs(mu(0,0)) > 100 && flipped == false){ // only flip "flipped" to true once, when the robot is close to the wall
    flipped = true;
  }
  if (flipped == true) {
    muflip = mu(0,0);
    analogWrite(12, 255);
    analogWrite(7, 255);
    analogWrite(13, 0); // forward
    analogWrite(6, 0);
    delay(15); // for currently unknown reasons, avoiding constantly writing new motor values seems to help my motors function correctly
  }
```

### Why the Flip is Theoretical
I have yet to actually acheive a flip with my robot. When I try to complete the stunt, my robot consistently turns instead of flipping. I think this is because of differing motor dynamics between the two motors on my robot. I believe that one motor is switching direction much more quickly than the other motor, which is causing the robot to turn rather than flip.

Below are some videos of the robot turning instead of flipping:

## Open Loop Stunt
[video 1](https://photos.app.goo.gl/m6nehSgzofdabECC9)
[video 2](https://photos.app.goo.gl/dF26o3prUiyg4TmT7)
[video 3](https://photos.app.goo.gl/EoGe5DcBscHbzFTU8)
Despite all the drifting and turns, the robot ends up in approximately the same spot each time.

## Blooper Videos
[it flips!!](https://photos.app.goo.gl/GfRHqqUpcPGycWCg7)


[life on the edge](https://photos.app.goo.gl/MQZ21gRPkQjnVQ5D8)