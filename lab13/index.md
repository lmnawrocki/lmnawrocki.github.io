# Lab 13

*coming soon...*
## The Plan
in a few parts

#### part 1 - heck localization
As you may have noticed, my lab 12 report was moderately unhinged and the localization was only slightly better than throwing a dart at the map.

Localization is also very slow--it takes time to run it on my computer even when optomized, in addition to the time it takes the robot to spin in a circle to collect the necessary data.

Also, even if the localization did work well, the added motion from spinning the robot changes its location and heading, which adds another way my robot can behave unpredictably.

Therefore, shockingly, I will not be doing any localization in this lab.

#### part 2 - wall following, sort of
Luckily, I still have two TOF sensors, which is good enough for doing some wall following shenanigans. I think that I can do some mediocre enough pretenting-to-be-localization with just my two TOF sensors and some odometry.

#### part 3 - the gyroscope
Please ignore all the mean things I said about my gyroscope in the last report. My gyroscope may or may not be living in a different coordinate system, but at least it's doing so consistently. Therefore, I'm going to use the gyroscope to perform the turns in this lab accurately-ish, and go through fine tuning it in order to perform the turns consistently.

#### part 4 - taking it slow
My robot will be easier to visually debug if I run it slowly, and this isn't a speed competition, so... slow.

