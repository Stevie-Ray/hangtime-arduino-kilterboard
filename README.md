# Fake Kilter Board
A fork of [1-max-1/fake_kilter_board](https://github.com/1-max-1/fake_kilter_board) ESP32 program to work with Arduino BLE and a Processing or [Web](https://grip-connect-kilter-board.vercel.app/?route=p1083r15p1117r15p1164r12p1185r12p1233r13p1282r13p1303r13p1372r13p1392r14p1505r15) (Chrome/Edge/Opera) desktop application working together to simulate a [Kilter Board](https://settercloset.com/pages/the-kilter-board). 

# Differences
- Processing: A Fullride (12 x 12 with kickboard) Kilterboard Layout

- A [web](https://grip-connect-kilter-board.vercel.app/?route=p1083r15p1117r15p1164r12p1185r12p1233r13p1282r13p1303r13p1372r13p1392r14p1505r15) application. **No Processing required**.

- Uses [ArduinoBLE](https://www.arduino.cc/reference/en/libraries/arduinoble/), this library is compatible with the **samd, megaavr, mbed, apollo3, mbed_nano, mbed_portenta, mbed_nicla, esp32, mbed_giga, renesas, renesas_portenta, mbed_opta, renesas_uno** architectures so you should be able to use it on the following Arduino boards:

    - Arduino MKR FOX 1200
    - Arduino MKR GSM 1400
    - Arduino MKR NB 1500
    - Arduino MKR VIDOR 4000
    - Arduino MKR WAN 1300 (LoRa connectivity)
    - Arduino MKR WAN 1310
    - Arduino MKR WiFi 1010
    - Arduino MKR ZERO (I2S bus & SD for sound, music & digital audio data)
    - Arduino MKR1000 WIFI
    - Arduino Nano 33 BLE
    - Arduino Nano 33 IoT
    - Arduino Nano Every
    - Arduino Uno WiFi REV2
    - Arduino Zero
    - Portenta H7
    - Arduino GIGA R1 WiFi
    - Arduino UNO R4 Minima
    - Arduino UNO R4 WiFi
    - Arduino Nano RP2040 Connect

![Kilterboard - Processing](https://github.com/user-attachments/assets/dc654efe-3a83-4a7b-b263-05f4af7846b4)

# Why would you want this
Maybe you're creating a project that interfaces with a kilter board, but you don't have one handy, or not one easily accessible during development (as was my case).
You can use this project to simulate one, then try it out on the actual thing once you've worked out the bugs.
Or maybe you just want to have a look through this project to get an idea of how the kilter/aurora board protocol functions.

# Installation and usage
This project uses [Processing](https://processing.org/) and [Arduino IDE](https://www.arduino.cc/en/software/), so get those if you haven't already.
1. Upload the `fakeAuroraBoard_arduino` project to the Arduino. Install the [ArduinoBLE](https://github.com/arduino-libraries/ArduinoBLE) library via **Tools > Manage Libraries...**.
2. Connect to one of the desktop applications:
    - For the Processing project (`fakeAuroraBoard_processing`), install the [BezierSQLib](https://github.com/fjenett/sql-library-processing) library via **Sketch > Import Library... > Manage Libraries...**. Click 'Run' and follow the prompts on the Processing window to connect it to the Arduino.
    - Or just connect to the [web](https://grip-connect-kilter-board.vercel.app/?route=p1083r15p1117r15p1164r12p1185r12p1233r13p1282r13p1303r13p1372r13p1392r14p1505r15) application.
5. Enjoy! Everything should now function as a regular Kilter Board - you can connect to it with the official app or your own software.

# Changing the board name
The name of aurora boards are in the following format:
1. A string of alphanumeric characters
2. Optionally followed by a serial number, which consts of a a `#` followed by digits.
3. Optionally followed by an API level which consists of a `@` character followed by *a* digit.
From what I've seen, API levels are either 2 or 3, with 3 being much more common now and 2 seems to be older.
If the API level is omitted, the kilter board app assumes API level 2.

For example, the following are all valid board names (except for the fact that the serial numbers are probably not actual serial numbers):

`mykilterboard#2353@3`,&nbsp;&nbsp;`mykilterboard@2`,&nbsp;&nbsp;`mykilterboard`,&nbsp;&nbsp;`mykilterboard#83727`

Each of these will show up in the app as `mykilterboard`.

If you want to change the name and/or api level of the fake board, look for `#define API_LEVEL` and `#define DISPLAY_NAME` in `fakeAuroraBoard_arduino.ino` in the arduino project.

# Using different kilter board models
A kilter board maps an (x, y) coordinate for a hold to a specific unique 'position' number, and each of the many different kilter board models seems to have their own set of position numbers. The `LEDPositionParser` class in `fakeAuroraBoard_processing` handles the mapping of position numbers to coordinates. It does this by querying the required information from the sqlite database in the `data` folder, which is the actual database I've extracted from the official kilter board app. This project is set up for a specific model and layout, so if you want to use a different one you may need to poke around in the database for the required ID's and then modify the SQL query accordingly in `LEDPositionParser`. Additionally, if the model you want to use is not the same size as the model in this project (17x19), then you will also need to modify the window size and draw loop to draw more squares.

### Grabbing board details from a registered gym
If the board you want to interface with happens to be in a gym that is registered on the official kilter board app, there is an easy way to get the required ID's. There is a backend API for the kilter board app and after intercepting a few requests I've identified an endpoint that can be used to return board details for a gym. Note that this will require a kilter board account. Just download the app and register (it's free).
1. Get the python script included in this repository.
2. Run `python3 getBoardDetails.py -g ` to list all of the registered gyms. You may wish to pipe this to a file.
3. Search for the JSON object that represents your gym, then copy the value in the `ID ` field.
4. Run `python3 getBoardDetails.py -u YourUsername -p YourPassword -i YourGymID ` to get a list of all the boards in the gym. This will also print out information such as the product, layout, size and set ID's for each board.

# The Aurora Board protocol
Note that I say "Aurora Board" here instead of "Kilter Board", as there are a few different companies (including the one that makes the kilter boards) that all use the same underlying software made by [Aurora Climbing](https://auroraclimbing.com/). Therefore, this information *should* apply (in theory) to all of these boards.

Aurora Boards communicate over BLE. They advertise a service with UUID of `4488B571-7806-4DF6-BCFF-A2897E4953FF`, this is how the corresponding app filters them.
Additionally, they contain a service with UUID of `6E400001-B5A3-F393-E0A9-E50E24DCCA9E`, which itself contains a characteristic with UUID of `6E400002-B5A3-F393-E0A9-E50E24DCCA9E`.
The app transfers a message to the board by writing data to this characteristic, in groups of 20 bytes, so it fits inside a bluetooth chunk.

A message tells the board which holds to light up and what color they should be. A message consists of smaller sub-messages joined together, which I'll refer to as "packets".
Each packet is at most 260 bytes. If new data will make the current packet go over 260 bytes, then the packet is ended, joined onto the current message and a new packet is created for the new data.
However, generally only one packet is needed, because as soon as you start sending big messages, it starts to become *very* slow. BLE isn't really that great for high data rate transmissions.
The official app limits the number of holds you can light up, probably for this reason among others.
The message is sent "first in first out" style. i.e. the first byte of the first packet is sent to the board first.

The format of a packet is fairly simple. The first byte is always a 1. The second byte is the size of the packet data, which we'll get to in a minute. The third byte is a checksum of the packet data, calculated as follows (java):
```java
int checksum(java.util.List<Integer> list) {
    int i = 0;
    for (Integer intValue : list) {
        i = (i + intValue.intValue()) & 255;
    }
    return (~i) & 255;
}
```
The fourth byte is a 2. After this, the **packet data** is appended. Then the final byte in the packet is a 3.

Now for the packet data. The format of the packet data differs depending on API level:

### API level 2
The first byte in the data is dependent on where the packet is in the message as a whole:
- If this packet is the first packet in the message, then this byte gets set to 78 (`N`).
- If this is the last packet in the message, this byte gets set to 79 (`O`).
- If it is in the middle, the byte gets set to 77 (`M`).
- If this packet is the _only_ packet in the message, the byte gets set to 80 (`P`). Note that this takes priority over the other conditions.

The rest of the packet data is made of groups of 2 bytes, with each group representing a hold. A hold requires a 10-bit position and a 24-bit rgb color. The lowest 8 bits of the position get put in the first byte of the group. The highest 2 bits of the position get put into the lowest 2 bits of the second byte of the group.
The other 6 bits of the second byte are filled with the rgb color, 2 bits for each of the R, G, B components, with the R bits occupying the highest 2 bits of the byte, the B bits occupying the lowest 2 bits of the byte and the G bits in the middle.

Obviously this does require each of the color components to be compressed, e.g. `0xFF` goes to `0b11`. This reduces your color choices down to 64.

### API level 3
The first byte in the data is dependent on where the packet is in the message as a whole:
- If this packet is the first packet in the message, then this byte gets set to 82 (`R`).
- If this is the last packet in the message, this byte gets set to 83 (`S`).
- If it is in the middle, the byte gets set to 81 (`Q`).
- If this packet is the _only_ packet in the message, the byte gets set to 84 (`T`). Note that this takes priority over the other conditions.

The rest of the packet data is made of groups of 3 bytes, with each group representing a hold. A hold requires a 16-bit position and a 24-bit rgb color. The lowest 8 bits of the position get put in the first byte of the group. The highest 8 bits of the position get put in the second byte of the group.
The third byte of the group is filled with the rgb color, 3 bits for the R and G components, 2 bits for the B component, with the 3 R bits occupying the high end of the byte and the 2 B bits in the low end (hence 3 G bits in the middle).

Obviously this does require each of the color components to be compressed, e.g. `0xFF` goes to `0b111` for red and green, `0xFF` goes to `0b11` for blue.

Note that to the best of my knowledge, this information is correct, but I did have to glean it through a bit of reverse engineering, so there could be mistakes.

# Using this project with other types of aurora boards
As mentioned previously, the kilter board appears to share the underlying aurora board software with multiple different types of boards including:
- Tension Board
- Grasshopper Board
- Decoy Board
- Touchstone Board

So you could probably get this project to work with them without too much tinkering. However, the databases will probably be different with regards to the wall models and hold placement numbers. Also the python script will not work until the backend host url is updated.
