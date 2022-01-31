# Lab 1 -The Artemis Board
## Setup 
I followed the instructions on [this](https://learn.sparkfun.com/tutorials/artemis-development-with-arduino?_ga=2.30055167.1151850962.1594648676-1889762036.1574524297&_gac=1.19903818.1593457111.Cj0KCQjwoub3BRC6ARIsABGhnyahkG7hU2v-0bSiAeprvZ7c9v0XEKYdVHIIi_-J-m5YLdDBMc2P_goaAtA4EALw_wcB) page, (also linked in the lab instructions).
Essentially, you need to add an additonal board manager of [this link](https://raw.githubusercontent.com/sparkfun/Arduino_Apollo3/main/package_sparkfun_apollo3_index.json), then go to Tools > Board > Boards manager then find the SparkFun Apollo3, download the latest version of the package, then select that board as the board in use.
## Example: Blink it Up
Code: example code "Blink" <br>
Simply download the code onto the Artemis and watch the built in LED blink :) <br>
[Video Link](https://drive.google.com/file/d/1yLsrXDoeahX1N06xKtilnDeAcdcOby7F/view?usp=sharing)
## Example: Serial
Code: example code "Example04_Serial" <br>
It's important to make certain that in the line 
`#define BAUD 57600       // any number, common choices: 9600, 115200, 230400, 921600`
defines the BAUD rate to a number that you can set it to in the Serial Montior. If the number defined here and the number in the bottom right of the Serial Monitor don't match, your computer will be expecting a different communication speed than the Artemis, and therefore the things that you see in the Serial Monitor will be gibberish. <br>
To open the Serial Monitor, go to Tools > Serial Montior in the Arduino IDE. <br>
This code prints anything the user sends to the Serial Monitor on the computer back at the user. <br>
[Video Link](https://drive.google.com/file/d/12capsugxCA_vEygTfkfhMVuvJ83upcOO/view?usp=sharing)
## Example: analogRead
it read temperature. cool.
## Example: MicrophoneOutput
i talk lol

## Other Stuff I Learned (Web Design):
Building a website is kind of hard for a MechE major who has barely touched github, has never used markdown before, and gets overwhelmed easily by lots of pages and advice on how to do the same thing in a lot of different ways. <br>
I'm pretty happy with the simple stuff I ended up with, but it took me longer than I'm willing to admit to get to this very basic site. <br>

Some web-design notes:
Make new pages with a folder in the main repository folder (where the first index.md is stored). Then put new index.md files in the folder. The link to the page will be `<main website>/<folder name>`.