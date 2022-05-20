# Lab 13

## The Plan
in a few parts

#### part 1 - heck localization
As you may have noticed, my lab 12 report was moderately unhinged and the localization was only slightly better than throwing a dart at the map.

Localization is also very slow--it takes time to run it on my computer even when optomized, in addition to the time it takes the robot to spin in a circle to collect the necessary data.

Also, even if the localization did work well, the added motion from spinning the robot changes its location and heading, which adds another way my robot can behave unpredictably.

Therefore, shockingly, I will not be doing any localization in this lab.

#### part 2 - wall following, sort of
Luckily, I still have two TOF sensors, which is good enough for doing some wall following shenanigans. I think that I can do some mediocre enough pretenting-to-be-localization with just my two TOF sensors and some odometry.

#### part 3 - the gyroscope
Please ignore all the mean things I said about my gyroscope in the last report. My gyroscope may or may not be living in a different coordinate system, but at least it's doing so consistently. Therefore, I'm going to use the gyroscope to perform the turns in this lab accurately-ish, and go through fine tuning it in order to perform the turns consistently.

#### part 4 - taking it slow
My robot will be easier to visually debug if I run it slowly, and this isn't a speed competition, so... slow.

## What actually happened, in many parts
Spoiler alert: this lab did not go according to plan.

### part one: my gyroscope does not solve my problems
[PID not working album](https://photos.app.goo.gl/j6FjEC11PU35ezYr7)

Above are some videos taken right around the same time showing that my PID behavior is really inconsistent. Sometimes it turns, sometimes it doesn't. Sometimes it overshoots, sometimes it doesn't. I can't figure out how to make sense of my gyroscope's behavior and use it to lead to consistent results. I have a low pass filter. While my gains cause it to turn slowly, I found that turning them too high led to significant overshoot that my robot would fail to correct for.


### part two: blind little buggin' DVD
After discussing that things weren't working with Johnathan and some other students in lab, I realized that I could get my robot to hit a lot of the waypoints just by employing a simple algorithm that tried to keep the robot a certain distance from the walls.

[Video 1 -- turning a lot](https://photos.app.goo.gl/2uv9FxAt4SDSbaiL7)

It's somewhat similar to bug algorithms in the way that it avoids walls and covers a large area of the map.

I also call it the DVD algorithm because it's somewhat similar to the [way that the DVD logo bounces around](https://bouncingdvdlogo.com/).

The algorithm has just a few steps:
1. Go Forward
2. Run side sensor. If there is a wall less than 30cm from the robot, turn a little bit away from the wall.
3. Sense if there is a wall less than 44cm in front of the robot, and if so, turn a lot to avoid the wall.


With some slightly different parameters, I was able to get some slightly better performance shown here:
[Video](https://photos.app.goo.gl/7wDYjxSW9f5FPsKQ9) The robot covers more waypoints faster in this video.

#### Discussion of this algorithm
This approach had several pros and cons. 

Some downsides of this approach are that the robot does not have much information about where it is on the map, and therefore it's impossible to know which waypoint the robot is at. Some poor localization may be possible to implement when the robot is not moving using the magnometer and readings from both TOF values, but this wouldn't work with the grid localization framework made in previous labs without significant modification. The localization would only work when the motors are not powered as the magnetic fields created by them would mess with the magnometer readings.

A useful feature of this approach is that the robot can start anywhere on the map, and it's likely to see success eventually regardless of location. There are some starting points that will lead to more success than others, but 

### part three: TOF sensors do everything, going backwards, somewhat sucessfully
[Video]()

### Part 4: adding P control on position for a little boost
I attempted to add some proportional control using the TOF values for both the forward movements and the corrections for being too close or too far from a wall. I think this was my best approach in theory given what I had, but I still could have improved a lot on what I implemented.

I think the best way that I could have improved would have been by using two Kalman filters to more accurately determine the position of my robot. 
One of the Kalman filters 
I chose not to implement this kind of Kalman filter as I knew it wouldn't be very useful to my success unless the other Kalman filter 

### Part 5 - A* planning
If I wanted to take this lab a step further in another direction, I could have implemented A\* planning. While this would have been mildly interesting to implement, I chose not to do it as it wasn't going to produce any interesting videos and it would have been very difficult to translate to the real robot.

A major downside of A\* planning/graph search algorithms is that they don't really take into account the dynamics of the real robot, they only consider with positons and path are ideal. Bringing the A\* path into reality would be very difficult and impractical in this case, as a lot of the issues in this lab come from having imperfect and unpredictable control of the robot.

## Appendices - code snippets

### angular PID

(1) This if-statement exists because sometimes I had inconsistent things happenening with the gyroscope drifting positively. It was drifting positively at inconsistent rates. This is supposed to prevent the robot from doing most of its movement until the gyroscope settles and gives the controller good data.

(2) This else pairs with the if-statement above. If my gyroscope does not sense that the robot is turning to the right, I turn the robot to the right very slowly so that the gyroscope has something to measure and use to reach the setpoint.
```cpp
void taskTwoPID(){
  getAngle();
  if (pitchlpf <  0.0){ //(1)
      err = -30-pitchlpf; // used a setpoint of 30 degrees, pitchlpf is current integrated pitch using low pass filter
      if (n < 99){
      pitches[n] = pitchlpf; // used for transmitting data for debugging
      n = n + 1;
    }
    else{
      pitches[99] = 0.0;
    }
    if(err < -10.0){ // 
      analogWrite(12, 0);
      analogWrite(6, 0);
      analogWrite(13, 80 - err);
      analogWrite(7, 80 - err);
      getAngle();
    }
    else if (err > 0.0){
      analogWrite(13, 0);
      analogWrite(7, 0);
      analogWrite(12, 80 + err);
      analogWrite(6, 80 + err);
      getAngle();
    }
    else{
      Serial.println("stopping");
      hardStopMovement();
      tasknum = 3;
      tx_characteristic_float.writeValue(0.0);
      delay(5);
      stopMovement();
      pausing = true;
      n =0;
      tasktimestamp = millis();
    }
  }
  else{ //(2)
      analogWrite(12, 0);
      analogWrite(6, 0);
      analogWrite(13, 90);
      analogWrite(7, 90);
  }
}
```

### part two
The code moves between calling the two functions below. turnLeft() is executed when the front TOF sensor detects a wall nearby.
```cpp
void wallFollow(){
  if(tasknum == 7){
    stopdist = 440;
  }
  readDistSide();
  if(distanceside < 300){
     analogWrite(12, 0);
     analogWrite(6, 0);
     analogWrite(13, 110);
     analogWrite(7, 80);
  }
  else{
    readDistFront();
    if(distancefront > stopdist){
      analogWrite(13, 60);
      analogWrite(6, 60);
      analogWrite(12, 0);
      analogWrite(7, 0);
    }
    else{
      analogWrite(13, 0);
      analogWrite(6, 0);
      analogWrite(12, 0);
      analogWrite(7, 0);
      tasknum = 8;
    }
  }
}

void turnLeft(){
  if(tasktimestamp + 1500 > millis()){
      analogWrite(12, 0);
      analogWrite(6, 0);
      analogWrite(13, 125);
      analogWrite(7, 125);
  }
  if(tasktimestamp + 1500 < millis()){
    hardStopMovement();
    tasknum = 7;
    resetGyro();
    pausing = true;
    tasktimestamp = millis();
  }
}
```