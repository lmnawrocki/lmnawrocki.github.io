# Lab 2 - Bluetooth

## Setup
I followed the instructions on the course website. In summarry, you install Python and pip and create a virtual environment in the folder where you want your code to be saved. Then you start a Jupyter server and run the provided code `ble_arduino.ino`
\
To make sure that the Artemis and computer are connected via Bluetooth, modify the file `connection.yaml` and make certain that the artemis address is the same as the address it prints to the serial montior. Make sure each spot in the address has two digits, even if it's a number less than 10. (i.e. if one part of the address is 7, change it to 07)
\
After importing the necessary libraries, the lines
```
# Get ArtemisBLEController object
ble = get_ble_controller()

# Connect to the Artemis Device
ble.connect()
```
will connect the computer to the Artemis.

## Task 1 - Echo
C++ code:
```
tx_estring_value.clear();
tx_estring_value.append(char_arr);

tx_characteristic_string.writeValue(tx_estring_value.c_str());

Serial.print("Sent back: ");
Serial.println(tx_estring_value.c_str());
```
Python Code:
```
ble.send_command(CMD.ECHO, "hey")
```

## Task 2 - Send Three Floats
C++ Code:
```
float float_a, float_b, float_c;
success = robot_cmd.get_next_value(float_a);
if (!success)
    return;

success = robot_cmd.get_next_value(float_b);
if (!success)
    return;

success = robot_cmd.get_next_value(float_c);
if (!success)
    return;

Serial.print("Three Floats: ");
Serial.print(float_a);
Serial.print(", ");
Serial.print(float_b);
Serial.print(", ");
Serial.println(float_c);
```
Python Code:
```
ble.send_command(CMD.SEND_THREE_FLOATS, "3.1|2.6|4.5")
```

## Task 3 - Notification Handler

Define the notification handler function:
```
def notification_handler(uuidstring, byte_array):
    global notification
    
    floatbytes = ble.bytearray_to_float(byte_array)
    notification = ble.receive_float(ble.uuid['RX_FLOAT'])
    
    if notification != floatbytes:
        return notification
    else:
        return False
```
Test to make sure that the function is working:
```
ble.start_notify(ble.uuid['RX_FLOAT'], notification_handler)
notification = 5.9

import asyncio
while True:
    print(notification)
    await asyncio.sleep(1)
```
## Task 4

[//]: <> (In your report, briefly explain the difference between the two approaches: Receive a float value in Python using receive_float() on a characteristic that is defined as BLEFloatCharactersitic in the Arduino side)

When you receive a float value in Python using receive_float(), the type float is maintained throughout the process. Additionally, this is more efficient than sending it as a string because it takes fewer bits to send a float than a string of the same length. 
\
The floats and strings also have different UUIDs for transmission.
\
The string also has a max message size, whereas the float will just round to some maximum number of digits, which is different from the max message size, and puts a different limit on the data that can be transmitted into one variable.
[//]: <> (Receive a float value in Python using receive_string() (and subsequently converting it to a float type in Python) on a characteristic that is defined as a BLECStringCharactersitic in the Arduino side)
\
However, this changes when you need to send a lot of floats at once. There is an overhead cost to sending a single piece of data, and this can make it more efficient to send things as a string if you have a lot of floats to send at once
