# Lab 2 - Bluetooth

## Setup
I followed the instructions on the course website. In summarry, you install Python and pip and create a virtual environment in the folder where you want your code to be saved. Then you start a Jupyter server and run the provided code `ble_arduino.ino`
\
To make sure that the Artemis and computer are connected via Bluetooth, modify the file `connection.yaml` and make certain that the artemis address is the same as the address it prints to the serial montior. Make sure each spot in the address has two digits, even if it's a number less than 10. (i.e. if one part of the address is 7, change it to 07)
\
After importing the necessary libraries, the lines
```py
# Get ArtemisBLEController object
ble = get_ble_controller()

# Connect to the Artemis Device
ble.connect()
```
will connect the computer to the Artemis.

## Task 1 - Echo
This is completeing the ECHO case of the handle_command() function in the code that is downloaded to the Artemis.
C++ code:
```cpp
tx_estring_value.clear(); //clear the EString where the data is stored
tx_estring_value.append(char_arr); //append the character array transmitted from the computer

tx_characteristic_string.writeValue(tx_estring_value.c_str()); //write this to the characteristic string

Serial.print("Sent back: "); //print to Serial Monitor
Serial.println(tx_estring_value.c_str()); // print the string
```
Python Code:
```py
ble.send_command(CMD.ECHO, "hey") //sends text to echo to the Artemis over bluetooth
```
Change the "hey" to change what the Artemis echos back to the Serial Monitor.

## Task 2 - Send Three Floats
This is completeing the SEND_THREE_FLOATS case of the handle_command() function in the code that is downloaded to the Artemis.
C++ Code:
```cpp
float float_a, float_b, float_c; //declare the float variables

success = robot_cmd.get_next_value(float_a); //make sure float a is received correctly
if (!success)
    return;

success = robot_cmd.get_next_value(float_b); //make sure float a is received correctly
if (!success)
    return;

success = robot_cmd.get_next_value(float_c); //make sure float a is received correctly
if (!success)
    return;

// print the floats to the Serial Monitor

Serial.print("Three Floats: ");
Serial.print(float_a);
Serial.print(", ");
Serial.print(float_b);
Serial.print(", ");
Serial.println(float_c);
```
Python Code to send the floats to the Artemis.
The | delimeters are important to separate the data.
```py
ble.send_command(CMD.SEND_THREE_FLOATS, "3.1|2.6|4.5")
```

## Task 3 - Notification Handler

Define the notification handler function:
```py
def notification_handler(uuidstring, byte_array):
    global notification // declare that notification is the same as the global variable so this function can update it
    
    floatbytes = ble.bytearray_to_float(byte_array)
    notification = ble.receive_float(ble.uuid['RX_FLOAT'])
    
    if notification != floatbytes:
        return True // this function doesn't need to return anything, but it's good to know if something updated or not if needed
    else:
        return False
```
This updates the global variable "nofication" if it is different from what it was last time the funtion was called.
Test to make sure that the function is working:
```py
ble.start_notify(ble.uuid['RX_FLOAT'], notification_handler) // start the notification handler
notification = 0 // initialize the global notification variable

import asyncio // import a library to sleep
while True:
    print(notification)
    await asyncio.sleep(1)
```
asyncio allows the computer not output the print line for a second while letting background callback tasks run so that the notification variable properly updates each second.
\
This code implements notification handler as a callback function.
\
This while loop would run forever, but the float stops incrementing at a certain number, due to constraints on the Artemis.
## Task 4

[//]: <> (In your report, briefly explain the difference between the two approaches: Receive a float value in Python using receive_float() on a characteristic that is defined as BLEFloatCharactersitic in the Arduino side)

When you receive a float value in Python using receive_float(), the type float is maintained throughout the process. Additionally, when sending a single number on its own, this is more efficient than sending it as a string because it takes fewer bits to send a float than a string of the same length. The floats and strings have different UUIDs for transmission and it takes a different number of bits to send a float and string of the same length.
\
However, if you need to send a lot of floats very fast or at once, it's actually more efficient to put them together in a string and send them in packets. This is because there is an overhead cost to telling the computer that a piece of data is a float or a string, and sending things in together in strings reduces that overhead cost.
