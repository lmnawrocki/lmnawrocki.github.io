# Lab 12 - Localization

## The simulation works :)
Running the provided code, I can determine that the simulation works well.

![simworks](../images/lab12_simworks.PNG)

You can see that it works in some areas better than others. There are always

The map is more symmetrical near the top, which makes it harder for the Bayes filter to determine where the robot is accurately.

## Implementing on the Artemis

## Implementing in Python
```py
async def perform_observation_loop(self, rot_vel=120):
        """Perform the observation loop behavior on the real robot, where the robot does  
        a 360 degree turn in place while collecting equidistant (in the angular space) sensor
        readings, with the first sensor reading taken at the robot's current heading. 
        The number of sensor readings depends on "observations_count"(=18) defined in world.yaml.
        
        Keyword arguments:
            rot_vel -- (Optional) Angular Velocity for loop (degrees/second)
                        Do not remove this parameter from the function definition, even if you don't use it.
        Returns:
            sensor_ranges   -- A column numpy array of the range values (meters)
            sensor_bearings -- A column numpy array of the bearings at which the sensor readings were taken (degrees)
                               The bearing values are not used in the Localization module, so you may return a empty numpy array
        """
        ble.start_notify(ble.uuid['RX_FLOAT'], self.notification_handler_float) # start the notification handler for data transmission for floats
        ble.start_notify(ble.uuid['RX_STRING'], self.notification_handler_string) # start the notification handler for data transmission for strings
        # i'm not actually using strings. i'm just going to cast it directly back to a float in python
        # i'm doing the two transmission channels to keep things seperate so that no confusion between a sensor range and sensor bearing can occur
        while(len(robot.sensor_bearings) < 18): #while we have less than 18 measuments (measurements per turn)
            ble.send_command(CMD.PING, "") # get the robot to perform one small turn
            a = len(robot.sensor_bearings) # see how many measurements exist currently
            b = 0 # initialize a variable that will always be less than a + 1
            while(b < a + 1): # while we are waiting for a new measurement
                b = len(robot.sensor_bearings) # update variable
                await asyncio.sleep(.1) # wait
        ble.send_command(CMD.STOPSPIN, "") # stop spinning
        return self.sensor_ranges, self.sensor_bearings # return data
```

## Results

not gonna lie, this really did not work all that well. :/