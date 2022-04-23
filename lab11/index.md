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

I changed the signature of this function from the given setup so that I would be able to get my code to work. I added inputting a state of the robot as the function needs
######
```py
correctobs = mapper.get_views(x, y, a)
    #prob_array = np.zeros(18)
    prob = 1
    for i in range(mapper.OBS_PER_CELL):
        #prob_array[o] = loc.gaussian(obs[0], correctobs[0], loc.sensor_sigma)
        prob = prob*loc.gaussian(obs[i], correctobs[i], loc.sensor_sigma)
    return prob
```

## update_step()
```py
for x in range(mapper.MAX_CELLS_X):
    for y in range(mapper.MAX_CELLS_Y):
        for a in range(mapper.MAX_CELLS_A):
            #prob_array = np.zeros(18)
            prob = sensor_model(loc.obs_range_data, x, y, a)
            loc.bel[x,y,a] = prob*loc.bel_bar[x, y, a]

# normalize so that all probabilities add up to 1:
loc.bel = loc.bel / np.sum(loc.bel)
```

## Videos of Bayes Filter Working
[video 1 - marginal distribution not visible]()
[video 2 - marginal distribution visible]()