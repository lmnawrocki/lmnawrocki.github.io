# Lab 2 - Bluetooth

## Setup
I followed the instructions on the course website. In summarry, you install Python and pip and create a virtual environment in the folder where you want your code to be saved. Then you start a Jupyter server and run the provided code `ble_arduino.ino`
\
To make sure that the Artemis and computer are connected via Bluetooth, modify the file `connection.yaml` and make certain that the artemis address is the same as the address it prints to the serial montior. Make sure each spot in the address has two digits, even if it's a number less than 10. (i.e. if one part of the address is 7, change it to 07)

## Task 1 - Echo

```
tx_estring_value.clear();
tx_estring_value.append(char_arr);

tx_characteristic_string.writeValue(tx_estring_value.c_str());

Serial.print("Sent back: ");
Serial.println(tx_estring_value.c_str());
```

## Task 2 - Send Three Floats

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

## Task 3 - Notification Handler

## Task 4

[//]: <> (In your report, briefly explain the difference between the two approaches: Receive a float value in Python using receive_float() on a characteristic that is defined as BLEFloatCharactersitic in the Arduino side Receive a float value in Python using receive_string() (and subsequently converting it to a float type in Python) on a characteristic that is defined as a BLECStringCharactersitic in the Arduino side)
