# Lab 11 - Localization Using Bayes Filter

## Citations/People I talked about this lab with
I got and gave some help with debugging and optimization from Syd on Saturday 4/23 in office hours.

## Why use a Bayes Filter?
As shown in the previous lab, odometry is a terrible and unreliable way to figure out where a robot is. Odometry is useless if the robot is moved by any external force at any point in its use, as an external movement will cause the odometry model to be completely wrong.

## What is a Bayes Filter?
A Bayes filter is a probabilistic model of a robot's location. In our case, we take a bunch of sensor measurements from spinning around in place, and compare those with known measurements from mapping to figure out where the robot is most likely to be. We assume all probability distributions are Gaussians to do this.

The Bayes filter can be summarized into an alorithm in a few steps:

For all possible states of the robot:

1. sum the probabilities that the robot is in each state given the previous state multiplied by the probability that the robot was in that previous state (prediction step)
2. multiply the probability that the sensor values correspond to each possible state by the value from step 1 (update step)

Then, normalize the probabilites that the robot is in any particular state such that they all sum up to 1 at the end of the update step.

## compute_control(cur_pose, prev_pose)
In this fuction, we find the difference between the actual current state and the actual previous state to figure out what the control input to the robot was. This is a helper function for `odom_motion_model(cur_pose, prev_pose, u)`.

```python
delta_rot_1 = np.array([mapper.normalize_angle(np.arctan((cur_pose[1]-prev_pose[1])/(cur_pose[0] - prev_pose[0]+.00001)) - prev_pose[2])])delta_trans = np.array([((cur_pose[1] - prev_pose[1])**2 + (cur_pose[0] -prev_pose[0])**2)**.5])
delta_rot_2 = np.array([mapper.normalize_angle(cur_pose[2] - prev_pose[2]- delta_rot_1)])
```
These calculations exist because our robot only travels in one dimension at a time in our model; it can only rotate or move forward or backward in the direction of its heading.

`delta_rot_1` is the initial rotation, the rotation that had to occur for the robot to travel in the direction that it did.

`delta_trans` is the distance that the robot moves on the map.

`delta_rot_2` is the rotation that occured bewteen `delta_rot_1` and the heading at the end of the movement.

## odom_motion_model(cur_pose, prev_pose, u)
This function finds the probabilities that the movement occured based on the current and previous poses. This is a helper function for `prediction_step(cur_odom, prev_odom)`.

```py
actual_u  = compute_control(cur_pose, prev_pose)
    
rot1e = u[0] - actual_u[0]
transe = u[1] - actual_u[1]
rot2e = u[2] - actual_u[2]

rot1p =loc.gaussian(rot1e, 0, loc.odom_rot_sigma) # assume a gaussian distribution
transp = loc.gaussian(transe, 0, loc.odom_trans_sigma)
rot2p = loc.gaussian(rot2e, 0, loc.odom_rot_sigma)

prob = rot1p*transp*rot2p # multiply these probabilities because we want to find the probability that they all happened in the same motion

return prob
```

## prediction_step(cur_odom, prev_odom)
This is the prediction step of the Bayes Filter.
```py
u = compute_control(cur_odom, prev_odom) # find the control input
    for x in range(mapper.MAX_CELLS_X): # for each cell
        for y in range(mapper.MAX_CELLS_Y):
            for a in range(mapper.MAX_CELLS_A):
                if loc.bel[x, y, a] > .0001: # if the probability is already super low, no need to recalculate it
                    cur_pose = mapper.from_map(x, y, a) # find the current pose for that cell
                    summed = 0 # reset summing variable
                    for xp in range(mapper.MAX_CELLS_X): # for each other cell
                        for yp in range(mapper.MAX_CELLS_Y):
                            for ap in range(mapper.MAX_CELLS_A):
                                prev_pose = mapper.from_map(xp, yp, ap) # find the previous pose corresponding to the other cell
                                prob = odom_motion_model(cur_pose, prev_pose , u) # find the probability the robot moved between these two cells
                                summed = summed + prob*loc.bel[xp, yp, ap] # sum the probability above for every possible previous cell
                    loc.bel_bar[x, y, a] = summed # when done summing, assign value to the current cell's spot in bel_bar
```
Note the if statement in the above code is not necessary for the code to run, but it is a huge optimization of this code.

The threshold for skipping the calculation is currently about 5 times lower than the uniform distribution probability.

## sensor_model(obs, x, y, a)
This is a helper function for `update_step()`.

I changed the signature of this function from the given setup so that I would be able to get my code to work. I added inputting a state of the robot as the function needs a state to determine what the correct sensor observations should be.

I also found that returning a single probability float instead of an array would be more efficient as it would mean that I wouldn't need to have another inner loop in the update step to multiply all the probabilities together.

```py
correctobs = mapper.get_views(x, y, a) # get the correct observations for this state
    prob = 1 # initialize probability variable
    for i in range(mapper.OBS_PER_CELL):
        prob = prob*loc.gaussian(obs[i], correctobs[i], loc.sensor_sigma) # multiply together probabilities of each sensor measurement
    return prob
```

## update_step()
In the update step, bel is updates based on the sensor model and bel_bar.
```py
for x in range(mapper.MAX_CELLS_X): # loop through every possible state
    for y in range(mapper.MAX_CELLS_Y):
        for a in range(mapper.MAX_CELLS_A):
            prob = sensor_model(loc.obs_range_data, x, y, a) # call the sensor model function to get the probability of the current sensor readings for the state in question
            loc.bel[x,y,a] = prob*loc.bel_bar[x, y, a] # modify bel accordingly

# normalize bel so that all probabilities add up to 1:
# this is especially important because some low probabilities were skipped in the prediction step
loc.bel = loc.bel / np.sum(loc.bel)
```

## Video of Bayes Filter Working
[video](https://drive.google.com/file/d/1PB8P3Wmk39q08hw8HZQzCwylzMT76uVH/view?usp=sharing)

[//]: # (Include the most probable state after each iteration of the bayes filter along with its probability and compare it with the ground truth pose. Write down your inference of when it works and when it doesn’t.)

The system works pretty well. It rounds to the center of the nearest grid cell, which causes some imperfections in the path.

## Optimization and Efficiency
Efficiency of this code matters quite a bit because there are so many calculations to do in an un-optomized time step, particularly in the prediction step.

While I made some effort to optomize my code, there is definitely room to improve and optomize it further. I could likely experiment with my threshold to skip cells, and see how threshold too high could break my predictions and cause them to be incorrect when the robot moves a lot or the sensor values change a lot.

I could also potentially optomize slightly by not assigning things that are only used once to variables.

Lastly, I could do some more things using matrices and vectors to optomize what I'm looping over further.

However, I am not a CS major and do not find this particularly interesting, so I have skipped this. There are definitely other ways to optimize this code that I am unaware of.

## Debug Output
```
----------------- 0 -----------------
2022-04-23 22:35:13,042 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:13,046 | INFO     |: GT index         : (6, 3, 6)
2022-04-23 22:35:13,047 | INFO     |: Prior Bel index  : (5, 3, 9) with prob = 0.0044057
2022-04-23 22:35:13,048 | INFO     |: POS ERROR        : (0.291, 0.213, -50.775)
2022-04-23 22:35:13,049 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:16,257 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:35:16,264 | INFO     |: GT index      : (6, 3, 6)
2022-04-23 22:35:16,265 | INFO     |: Bel index     : (6, 4, 6) with prob = 1.0
2022-04-23 22:35:16,266 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:35:16,267 | INFO     |: GT            : (0.291, -0.092, 319.225)
2022-04-23 22:35:16,268 | INFO     |: Belief        : (0.305, 0.000, -50.000)
2022-04-23 22:35:16,268 | INFO     |: POS ERROR     : (-0.014, -0.092, 369.225)
2022-04-23 22:35:16,269 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 1 -----------------
2022-04-23 22:35:18,402 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:18,405 | INFO     |: GT index         : (7, 2, 5)
2022-04-23 22:35:18,406 | INFO     |: Prior Bel index  : (6, 4, 6) with prob = 0.0083484
2022-04-23 22:35:18,407 | INFO     |: POS ERROR        : (0.202, -0.543, 345.542)
2022-04-23 22:35:18,409 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:21,608 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:35:21,610 | INFO     |: GT index      : (7, 2, 5)
2022-04-23 22:35:21,611 | INFO     |: Bel index     : (6, 2, 5) with prob = 1.0
2022-04-23 22:35:21,612 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:35:21,613 | INFO     |: GT            : (0.507, -0.543, 655.542)
2022-04-23 22:35:21,614 | INFO     |: Belief        : (0.305, -0.610, -70.000)
2022-04-23 22:35:21,614 | INFO     |: POS ERROR     : (0.202, 0.067, 725.542)
2022-04-23 22:35:21,615 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 2 -----------------
2022-04-23 22:35:22,758 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:22,765 | INFO     |: GT index         : (7, 2, 4)
2022-04-23 22:35:22,766 | INFO     |: Prior Bel index  : (6, 4, 6) with prob = 0.0083484
2022-04-23 22:35:22,767 | INFO     |: POS ERROR        : (0.202, -0.543, 681.858)
2022-04-23 22:35:22,768 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:25,978 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:35:25,996 | INFO     |: GT index      : (7, 2, 4)
2022-04-23 22:35:25,997 | INFO     |: Bel index     : (7, 2, 4) with prob = 1.0
2022-04-23 22:35:25,998 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:35:25,999 | INFO     |: GT            : (0.507, -0.543, 991.858)
2022-04-23 22:35:26,000 | INFO     |: Belief        : (0.610, -0.610, -90.000)
2022-04-23 22:35:26,000 | INFO     |: POS ERROR     : (-0.103, 0.067, 1081.858)
2022-04-23 22:35:26,001 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 3 -----------------
2022-04-23 22:35:27,126 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:27,129 | INFO     |: GT index         : (7, 0, 4)
2022-04-23 22:35:27,130 | INFO     |: Prior Bel index  : (6, 4, 6) with prob = 0.0083484
2022-04-23 22:35:27,130 | INFO     |: POS ERROR        : (0.215, -0.949, 1041.858)
2022-04-23 22:35:27,132 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:30,357 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:35:30,373 | INFO     |: GT index      : (7, 0, 4)
2022-04-23 22:35:30,374 | INFO     |: Bel index     : (6, 1, 4) with prob = 0.9999999
2022-04-23 22:35:30,375 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:35:30,376 | INFO     |: GT            : (0.520, -0.949, 1351.858)
2022-04-23 22:35:30,377 | INFO     |: Belief        : (0.305, -0.914, -90.000)
2022-04-23 22:35:30,378 | INFO     |: POS ERROR     : (0.215, -0.035, 1441.858)
2022-04-23 22:35:30,379 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 4 -----------------
2022-04-23 22:35:33,529 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:33,546 | INFO     |: GT index         : (8, 0, 8)
2022-04-23 22:35:33,546 | INFO     |: Prior Bel index  : (6, 4, 6) with prob = 0.0083484
2022-04-23 22:35:33,547 | INFO     |: POS ERROR        : (0.479, -1.101, 1489.233)
2022-04-23 22:35:33,548 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:36,770 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:35:36,779 | INFO     |: GT index      : (8, 0, 8)
2022-04-23 22:35:36,779 | INFO     |: Bel index     : (8, 1, 9) with prob = 0.9999999
2022-04-23 22:35:36,781 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:35:36,782 | INFO     |: GT            : (0.784, -1.101, 1799.233)
2022-04-23 22:35:36,783 | INFO     |: Belief        : (0.914, -0.914, 10.000)
2022-04-23 22:35:36,785 | INFO     |: POS ERROR     : (-0.130, -0.187, 1789.233)
2022-04-23 22:35:36,786 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 5 -----------------
2022-04-23 22:35:42,933 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:42,942 | INFO     |: GT index         : (11, 0, 11)
2022-04-23 22:35:42,943 | INFO     |: Prior Bel index  : (6, 4, 6) with prob = 0.0083484
2022-04-23 22:35:42,944 | INFO     |: POS ERROR        : (1.278, -0.948, 1899.022)
2022-04-23 22:35:42,946 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:46,152 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:35:46,160 | INFO     |: GT index      : (11, 0, 11)
2022-04-23 22:35:46,161 | INFO     |: Bel index     : (10, 1, 11) with prob = 1.0
2022-04-23 22:35:46,162 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:35:46,163 | INFO     |: GT            : (1.583, -0.948, 2209.022)
2022-04-23 22:35:46,164 | INFO     |: Belief        : (1.524, -0.914, 50.000)
2022-04-23 22:35:46,165 | INFO     |: POS ERROR     : (0.059, -0.033, 2159.022)
2022-04-23 22:35:46,165 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 6 -----------------
2022-04-23 22:35:48,299 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:48,316 | INFO     |: GT index         : (11, 2, 12)
2022-04-23 22:35:48,317 | INFO     |: Prior Bel index  : (6, 4, 6) with prob = 0.0083484
2022-04-23 22:35:48,318 | INFO     |: POS ERROR        : (1.362, -0.550, 2288.153)
2022-04-23 22:35:48,319 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:51,511 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:35:51,523 | INFO     |: GT index      : (11, 2, 12)
2022-04-23 22:35:51,524 | INFO     |: Bel index     : (10, 2, 12) with prob = 1.0
2022-04-23 22:35:51,525 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:35:51,526 | INFO     |: GT            : (1.666, -0.550, 2598.153)
2022-04-23 22:35:51,527 | INFO     |: Belief        : (1.524, -0.610, 70.000)
2022-04-23 22:35:51,527 | INFO     |: POS ERROR     : (0.142, 0.060, 2528.153)
2022-04-23 22:35:51,529 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 7 -----------------
2022-04-23 22:35:53,660 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:53,663 | INFO     |: GT index         : (11, 3, 13)
2022-04-23 22:35:53,664 | INFO     |: Prior Bel index  : (6, 4, 6) with prob = 0.0083484
2022-04-23 22:35:53,665 | INFO     |: POS ERROR        : (1.437, -0.191, 2653.979)
2022-04-23 22:35:53,666 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:35:56,856 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:35:56,873 | INFO     |: GT index      : (11, 3, 13)
2022-04-23 22:35:56,874 | INFO     |: Bel index     : (11, 3, 13) with prob = 1.0
2022-04-23 22:35:56,875 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:35:56,876 | INFO     |: GT            : (1.741, -0.191, 2963.979)
2022-04-23 22:35:56,877 | INFO     |: Belief        : (1.829, -0.305, 90.000)
2022-04-23 22:35:56,878 | INFO     |: POS ERROR     : (-0.087, 0.113, 2873.979)
2022-04-23 22:35:56,879 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 8 -----------------
2022-04-23 22:36:00,023 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:00,034 | INFO     |: GT index         : (11, 5, 14)
2022-04-23 22:36:00,035 | INFO     |: Prior Bel index  : (6, 4, 6) with prob = 0.0083484
2022-04-23 22:36:00,036 | INFO     |: POS ERROR        : (1.438, 0.317, 3036.998)
2022-04-23 22:36:00,038 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:03,253 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:36:03,258 | INFO     |: GT index      : (11, 5, 14)
2022-04-23 22:36:03,259 | INFO     |: Bel index     : (11, 4, 13) with prob = 0.6380162
2022-04-23 22:36:03,260 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:36:03,261 | INFO     |: GT            : (1.743, 0.317, 3346.998)
2022-04-23 22:36:03,262 | INFO     |: Belief        : (1.829, 0.000, 90.000)
2022-04-23 22:36:03,262 | INFO     |: POS ERROR     : (-0.086, 0.317, 3256.998)
2022-04-23 22:36:03,266 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 9 -----------------
2022-04-23 22:36:06,497 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:06,515 | INFO     |: GT index         : (11, 6, 16)
2022-04-23 22:36:06,515 | INFO     |: Prior Bel index  : (6, 4, 6) with prob = 0.0083484
2022-04-23 22:36:06,516 | INFO     |: POS ERROR        : (1.441, 0.652, 3437.769)
2022-04-23 22:36:06,519 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:09,726 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:36:09,739 | INFO     |: GT index      : (11, 6, 16)
2022-04-23 22:36:09,741 | INFO     |: Bel index     : (11, 7, 16) with prob = 1.0
2022-04-23 22:36:09,742 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:36:09,743 | INFO     |: GT            : (1.746, 0.652, 3747.769)
2022-04-23 22:36:09,745 | INFO     |: Belief        : (1.829, 0.914, 150.000)
2022-04-23 22:36:09,746 | INFO     |: POS ERROR     : (-0.083, -0.262, 3597.769)
2022-04-23 22:36:09,747 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 10 -----------------
2022-04-23 22:36:11,880 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:11,897 | INFO     |: GT index         : (10, 7, 16)
2022-04-23 22:36:11,898 | INFO     |: Prior Bel index  : (6, 4, 6) with prob = 0.0083484
2022-04-23 22:36:11,899 | INFO     |: POS ERROR        : (1.011, 0.924, 3809.421)
2022-04-23 22:36:11,900 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:15,134 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:36:15,150 | INFO     |: GT index      : (10, 7, 16)
2022-04-23 22:36:15,151 | INFO     |: Bel index     : (9, 7, 17) with prob = 1.0
2022-04-23 22:36:15,152 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:36:15,153 | INFO     |: GT            : (1.316, 0.924, 4119.422)
2022-04-23 22:36:15,155 | INFO     |: Belief        : (1.219, 0.914, 170.000)
2022-04-23 22:36:15,155 | INFO     |: POS ERROR     : (0.097, 0.009, 3949.422)
2022-04-23 22:36:15,157 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 11 -----------------
2022-04-23 22:36:18,303 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:18,305 | INFO     |: GT index         : (7, 6, 3)
2022-04-23 22:36:18,306 | INFO     |: Prior Bel index  : (6, 4, 6) with prob = 0.0083484
2022-04-23 22:36:18,308 | INFO     |: POS ERROR        : (0.107, 0.780, 4267.760)
2022-04-23 22:36:18,310 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:21,529 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:36:21,542 | INFO     |: GT index      : (7, 6, 3)
2022-04-23 22:36:21,543 | INFO     |: Bel index     : (7, 7, 3) with prob = 0.9999381
2022-04-23 22:36:21,543 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:36:21,544 | INFO     |: GT            : (0.412, 0.780, 4577.761)
2022-04-23 22:36:21,545 | INFO     |: Belief        : (0.610, 0.914, -110.000)
2022-04-23 22:36:21,546 | INFO     |: POS ERROR     : (-0.197, -0.134, 4687.761)
2022-04-23 22:36:21,547 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 12 -----------------
2022-04-23 22:36:23,666 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:23,668 | INFO     |: GT index         : (6, 4, 6)
2022-04-23 22:36:23,669 | INFO     |: Prior Bel index  : (6, 4, 6) with prob = 0.0083484
2022-04-23 22:36:23,670 | INFO     |: POS ERROR        : (-0.033, 0.134, 4673.607)
2022-04-23 22:36:23,671 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:26,902 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:36:26,918 | INFO     |: GT index      : (6, 4, 6)
2022-04-23 22:36:26,919 | INFO     |: Bel index     : (6, 4, 6) with prob = 1.0
2022-04-23 22:36:26,920 | INFO     |: Bel_bar prob at index = 0.008348448576207906
2022-04-23 22:36:26,921 | INFO     |: GT            : (0.272, 0.134, 4983.607)
2022-04-23 22:36:26,922 | INFO     |: Belief        : (0.305, 0.000, -50.000)
2022-04-23 22:36:26,923 | INFO     |: POS ERROR     : (-0.033, 0.134, 5033.607)
2022-04-23 22:36:26,924 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 13 -----------------
2022-04-23 22:36:29,051 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:29,053 | INFO     |: GT index         : (6, 3, 2)
2022-04-23 22:36:29,054 | INFO     |: Prior Bel index  : (5, 3, 9) with prob = 0.0044057
2022-04-23 22:36:29,055 | INFO     |: POS ERROR        : (0.032, 0.111, 4903.718)
2022-04-23 22:36:29,056 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:32,295 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:36:32,303 | INFO     |: GT index      : (6, 3, 2)
2022-04-23 22:36:32,304 | INFO     |: Bel index     : (5, 3, 2) with prob = 1.0
2022-04-23 22:36:32,305 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:36:32,306 | INFO     |: GT            : (0.032, -0.193, 5273.718)
2022-04-23 22:36:32,307 | INFO     |: Belief        : (0.000, -0.305, -130.000)
2022-04-23 22:36:32,308 | INFO     |: POS ERROR     : (0.032, 0.111, 5403.718)
2022-04-23 22:36:32,309 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 14 -----------------
2022-04-23 22:36:35,463 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:35,477 | INFO     |: GT index         : (4, 2, 1)
2022-04-23 22:36:35,478 | INFO     |: Prior Bel index  : (5, 3, 9) with prob = 0.0044057
2022-04-23 22:36:35,478 | INFO     |: POS ERROR        : (-0.338, -0.058, 5240.413)
2022-04-23 22:36:35,479 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:38,682 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:36:38,698 | INFO     |: GT index      : (4, 2, 1)
2022-04-23 22:36:38,699 | INFO     |: Bel index     : (4, 3, 1) with prob = 1.0
2022-04-23 22:36:38,700 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:36:38,701 | INFO     |: GT            : (-0.338, -0.363, 5610.413)
2022-04-23 22:36:38,702 | INFO     |: Belief        : (-0.305, -0.305, -150.000)
2022-04-23 22:36:38,703 | INFO     |: POS ERROR     : (-0.033, -0.058, 5760.413)
2022-04-23 22:36:38,704 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------


----------------- 15 -----------------
2022-04-23 22:36:41,844 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:41,858 | INFO     |: GT index         : (3, 2, 0)
2022-04-23 22:36:41,859 | INFO     |: Prior Bel index  : (5, 3, 9) with prob = 0.0044057
2022-04-23 22:36:41,860 | INFO     |: POS ERROR        : (-0.738, -0.067, 5577.108)
2022-04-23 22:36:41,861 | INFO     |: ---------- PREDICTION STATS -----------
2022-04-23 22:36:45,085 | INFO     |: ---------- UPDATE STATS -----------
2022-04-23 22:36:45,096 | INFO     |: GT index      : (3, 2, 0)
2022-04-23 22:36:45,097 | INFO     |: Bel index     : (3, 3, 0) with prob = 0.9965666
2022-04-23 22:36:45,098 | INFO     |: Bel_bar prob at index = 0.00051440329218107
2022-04-23 22:36:45,099 | INFO     |: GT            : (-0.738, -0.372, 5947.108)
2022-04-23 22:36:45,099 | INFO     |: Belief        : (-0.610, -0.305, -170.000)
2022-04-23 22:36:45,100 | INFO     |: POS ERROR     : (-0.129, -0.067, 6117.108)
2022-04-23 22:36:45,101 | INFO     |: ---------- UPDATE STATS -----------
-------------------------------------
```