# Lab 13
*the finale...*

## The Plan
###### (in a few parts)

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

See [here](https://lmnawrocki.github.io/lab13/#angular-pid) for some code snippets and discussion on the code.

All of the bad things I said about gyroscopes in previous labs are re-instated.

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
[Video 2 -- turning a bit less](https://photos.app.goo.gl/7wDYjxSW9f5FPsKQ9) The robot covers more waypoints faster in this video.

See [here](https://lmnawrocki.github.io/lab13/#part-two) for code snippets and discussion of the code details.

#### Discussion of this algorithm
This approach had several pros and cons. 

One downside of this approach are that the robot does not have much information about where it is on the map, and therefore it's impossible to know which waypoint the robot is at. This is a failure to complete one of the important tasks in this lab. Some poor localization may be possible to implement when the robot is not moving using the magnometer and readings from both TOF values, but this wouldn't work with the grid localization framework made in previous labs without significant modification. The localization would only work when the motors are not powered as the magnetic fields created by them would mess with the magnometer readings.

Another downside of this approach is that I did not implement anything to prevent the robot from getting stuck in certain cases. This can be seen at the end of the videos when the robot does in fact get stuck. *sigh*

I could have avoided the robot getting stuck by adding some code where if the 5 most recent TOF values were very close to one another, the robot could initiate some kind of spinning sequence at full power to make the chances that it would become un-stuck very high.

A useful feature of this approach is that the robot can start anywhere on the map, and it's likely to see success eventually regardless of location. There are some starting points that will lead to more success than others, but if the robot doesn't get stuck, it should reach every point eventually.

### part three: TOF sensors do everything, going backwards, somewhat sucessfully
[Here's my most sucessful video of this approach](https://photos.app.goo.gl/vMpu9QdHnpsBrvMD6)
As you can see, this approach is incredibly slow. I ran it very slowly because there were two TOF sensor readings in each time step. This caused a .2s delay before the robot reacted to its surroundings. The robot consistently hit the wall due to its slow reaction time if it went much faster.

Additionally, the robot does not manage to move away from the wall in the second part of the video. This is because I was not able to tune the turning well to have it turn away from the wall in this case. I found that using powerful turns to move away from a wall led to turning too much and hitting the other wall.

### Part 4: adding P control on position for a little boost
[Here's the most successful video of this approach](https://photos.app.goo.gl/k5otHEfbTgyuJh7e6)
I attempted to add some proportional control using the TOF values for both the forward movements and the corrections for being too close or too far from a wall. I think this was my best approach in theory given what I had, but I still could have improved a lot on what I implemented.

I think the best way that I could have improved would have been by using two Kalman filters to more accurately determine the position of my robot. 
One of the Kalman filters 
I chose not to implement this kind of Kalman filter as I knew it wouldn't be very useful to my success unless the other Kalman filter was also implemented.

I think an implementation of both Kalman filters would be an interesting substitute for localization. In theory, if both Kalman filters were working well 

Here's a [blooper](https://photos.app.goo.gl/Dz1bi6Zn5VyMBm5x6) that I thought was rather interesting, especially in the way the robot drifted in the end. It shows that the P controller for being away from the walls works rather well, as information that the robot is getting saying that it's far from walls is causing it to try to spin closer to the wall, but it fails gracefully.

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
    stopdist = 440; // threshold for robot performing a big turn. tuning this parameter bigger or smaller will help the robot hit waypoints different distances from the walls. I found this worked well for these waypoints and this map, since this is the approximate distance between many of the waypoints and a wall.
  }
  readDistSide(); // check the side sensor first, since not hitting a wall on the side is the primary way the robot gets stuck.
  if(distanceside < 300){ // this parameter is smaller than the other one because if the robot is close to the wall and near a waypoint, I don't want to cause the robot to miss that waypoint
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
      tasknum = 8; // switch to a big turn
    }
  }
}

void turnLeft(){
  if(tasktimestamp + 1500 > millis()){ // OL timer-based control for turning due to my gyroscope issues
      analogWrite(12, 0);
      analogWrite(6, 0);
      analogWrite(13, 125);
      analogWrite(7, 125);
  }
  if(tasktimestamp + 1500 < millis()){
    hardStopMovement();
    tasknum = 7; // big turn is completed, switch back to the other part of the algorithm
    resetGyro();
    pausing = true; // I pause for a small amount of time between turning to let the robot stop moving before applying a new control. this makes the dynamics of my robot more predictable and intuitive to debug
    tasktimestamp = millis();
  }
}
```

### P control on position
```cpp
void pauseBetweenTasks(){
  if(tasktimestamp + 1000 > millis()){
    hardStopMovement();
  }
  else{
    pausing = false;
  }
}

void taskOne(){
  if(tasktimestamp + 400 > millis()){
    goForward();
  }
  if(tasktimestamp + 400 < millis()){
    hardStopMovement();
    tasknum = 2;
    resetGyro();
    pausing = true;
    distanceSensor2.startRanging();
    distanceSensor1.startRanging();
    tasktimestamp = millis();
  }
}

void wallFollow(){
  if(tasknum == 3 || tasknum == 4 || tasknum == 5){
    stopdist = 500;
    sidedist = 300;
    checkDistFront();
    checkDistSide();
  }
  if (tasknum == 2){
    sidedist = 550;
    stopdist = 400;
    distanceside = 550;
    checkDistFront();
  }
  if(newdistfront == true){
    if(distancefront < stopdist){
      hardStopMovement();
      tasktimestamp = millis();
      if(tasknum == 2){
        tasknum = 3;
        tx_characteristic_float.writeValue(2.0);
        delay(5);
        turning = true;
        pausing = true;
      }
      else if (tasknum == 3){
        tasknum =4;
        tx_characteristic_float.writeValue(3.0);
        delay(5);
        turning = true;
        pausing = true;
      }
      else if(tasknum == 4){
        tasknum =5;
        tx_characteristic_float.writeValue(4.0);
        delay(5);
        turning = true;
        pausing = true;
      }
      else if (tasknum == 5){
        tasknum =6;
        tx_characteristic_float.writeValue(5.0);
        delay(5);
        turning = true;
        pausing = true;
      }
      tasktimestamp = millis();
    }
    else if(newdistside == true){
      if(distanceside < sidedist - 50){
        err = ((sidedist-50) - distanceside)/20;
        analogWrite(12, 0);
        analogWrite(6, 10 + err);
        analogWrite(13, 100 + err);
        analogWrite(7, 0);
        tx_characteristic_float.writeValue(.3);
      }
      else if(distanceside > sidedist + 75){
        err = (-(sidedist+100) + distanceside)/20;
        analogWrite(12, 0);
        analogWrite(6, 100 + err);
        analogWrite(13, 10 + err);
        analogWrite(7, 0);
        tx_characteristic_float.writeValue(.4);
      }
      else {
        analogWrite(13, 60);
        analogWrite(6, 60);
        analogWrite(12, 0);
        analogWrite(7, 0);
      }
    }
    else{
      // proceed forward with P control
      err = (distancefront - stopdist)/10;
      if (err > 65){
        err = 65;
      }
      analogWrite(13, 60 + err);
      analogWrite(6, 60 + err);
      analogWrite(12, 0);
      analogWrite(7, 0);
    }
    tasktimestamp = millis();
  }
  tasktimestamp = millis();
}

void turnRight(){
  tx_characteristic_float.writeValue(.2);
  delay(5);
  if (first == true){
    analogWrite(13, 125);
    analogWrite(7, 125);
    analogWrite(12, 0);
    analogWrite(6, 0);
    first = false;
    tasktimestamp = millis();
  }
  if(tasktimestamp + 1000 > millis()){
      analogWrite(13, 125);
      analogWrite(7, 125);
      analogWrite(12, 0);
      analogWrite(6, 0);
  }
  if(tasktimestamp + 1000 < millis()){
    hardStopMovement();
    pausing = true;
    turning = false;
    first = true;
    distanceSensor2.startRanging();
    distanceSensor1.startRanging();
    tx_characteristic_float.writeValue(.1);
    tasktimestamp = millis();
  }
}

void sillyLittleTasks(){
  Serial.println(tasknum);
  if (pausing == true){
    pauseBetweenTasks();
  }
  else if(turning == true){
    turnRight();
  }
  else{
    if (tasknum == 0){
      tasknum = 1;
      tx_characteristic_float.writeValue(1.0);
      delay(5);
    }
    if (tasknum == 1){
      taskOne();
    }
    else{
      if (tasknum == 2 || tasknum==3 || tasknum == 4 || tasknum == 5){
        tx_characteristic_float.writeValue(.5);
         wallFollow();
      }
    }
  }
}

```