*coming soon...*
(send help)

# Lab 9 - Mapping
#### Shout-outs - Help Received
I talked to Priyam a bit about his approach to the lab and how to collect data.

### Some Opinions on Taping Wheels
I attempted to put tape over the wheels of my car, like many others have, to help the car spin on axis with less friction. Tape is really great to get the car to *start* spinning. However, slippery tape on the wheels is not so useful when I want my car to *stop* spinning.

I determined that while robots in this class are fast, this lab is not a speed competition, and it's very important that my robot stops in a consistent way so that I can get accurate distance measurements.

Therefore, my wheels do not have tape on them.

They do, however, have the sticky remnants of some tape, which I believe is the ideal situation.

### Some Bad Data
Here is a not great run of trying to get a map at location 5,3. Yikes!
![yikes](../images/yikes!.png)

This was a first attempt, but shows that I do not need so many data points. The robot went over one rotation and was also likely going much faster than I needed it to, since I was spinning it at full power.

### Open Loop Control
While the instructions for this lab claim that PID control is superior, in this situation, I disagree. The goal is for the robot to move in a way such that its end locations can be accurately measured. It does not matter so much if the measurement is taken at, say, 20 degrees or 21 degrees as long as we are reasonably confident in the sensor value. (ahahaha why would my sensor ever be wrong... not in this lab :woozy_face:)

Anyway, PID control is a lot of work to implement properly. Doing so does not exactly lead to better data. So, I used open loop control for this lab.

### Map Set-up (matlab)
```m
angles53 = angles53.*pi/180;
dist53 = dist53.*-1/1000;
polarplot(angles53,dist53,'.')
for i = 1:40
    x53(i) = dist53(i)*cos(angles53(i))+1.5;
    y53(i) = dist53(i)*sin(angles53(i))+1;
end
figure(2)
hold on
plot([-1.6764,-1.6764, -1.6764, 1.9812, -1.6764,1.9812,1.9812,-0.7620, -0.7620, -0.7620,-1.6764], [0.1524,-1.3716, -1.3716,-1.3716,-1.3716,-1.3716, 1.3716, 1.3716, 1.3716, 0.1524, 0.1524],'k', LineWidth=1)
plot([0.7620, 1.3716, 1.3716, 1.3716, 1.3716, 0.7620, 0.7620, 0.7620],[ -0.1524, -0.1524, -0.1524, 0.4572, 0.4572, 0.4572, 0.4572, -0.1524],'k', LineWidth=1)
plot([-0.1524, -0.1524, -0.1524, 0.1524, 0.1524, 0.1524],[-1.3716, -0.7620, -0.7620 -0.7620, -0.7620 ,-1.3716] ,'k', LineWidth=1)
xlim([-2,2])
ylim([-2,2])
axis equal
plot(x53, y53,'k.')
```
