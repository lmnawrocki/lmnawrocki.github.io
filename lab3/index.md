## Lab 3 Sensors

## Soldering stuff
# Why You should use shutdown pins
using shutdown pins will make it more clear in my code which TOF sensor the code is "talking" to at any point in time.

I am not the best at CS and thus it is good to know very clearly what is happening.

Also, I'm too lazy to think of a new address for the second TOF sensor, and this is a great way to avoid that.

(There's no way that's better than the other, really. Both ways work. Except my way is obviously better.)

# Set up TOF
Wire yellow to 
Wire blue to
Wire black to ground.
Wire red to V

I used pin 4 and 8 and green wires to connect the shutdown pins. (XSHUT on the TOF sensor.)

# Set up IMU
Wire yellow to 
Wire blue to
Wire black to ground.
Wire red to V

# some soldering stuff I learned
Soldering is very hard. The wires in this project are made up of a bunch of smaller wires. 
1. after stripping a wire, twist the end together to make it stronger
2. put a little bit of solder on the twisted wire. You don't need enough to make the wire noticibly thicker, just get it in between the threads and ideally down inside the coating.
3. CLEAN YOUR SOLDERING IRON! This may require tin and/or a wet sponge.
4. You can undo solder with suction or with a special kind of copper wire.

## 3a
1. When you just have one TOF, it prints one address. When you have two TOFs connected, it prints *every* address. This is not exactly what I expected, however, the TAs have confirmed this is normal behavior.

2. Setting the distance mode changes the maximum distnce the TOF can read. Longer distance modes can be less accurate, though.

I think that on the final robot, since it is so fast, it might be best to use the 1.3m mode since that can potentially tell the robot most accurately when it is about to hit a wall. I don't think it's as important for the robot to know how far away it is from far away things, but I could be wrong.

## 3b
AD0_VAL is the value of the last bit of the I2C address. It should be 0 in our case.

# Accelerometer


# Code Citations:
I used the Feb. 3 lecture slides for code for the complementary filter.