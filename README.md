# Continuous In-the-wild Smartwatch Activity Data

This dataset contains smartwatch sensor and location data collected in real-world settings. The data in this file is collected using the methods described in the following paper:

```D. Cook and M. Schmitter-Edgecombe. Fusing ambient and mobile sensor features into a behaviorome for prediction clinical health scores. IEEE Access, 2:65033-65043, 2021.```

When you use this dataset, we request that you cite this paper.

## Data Collection

Data were continuously collected from 49 participants while they performed their normal daily routines.
- Participants wore an Apple Watch on their wrist running a custom app which collected sensor data and prompt responses from the participant
- Participants wore the watch for varying amounts of time, ranging from one week to a month or longer
- Participants wore the watch while awake, and charged it while they were sleeping (see the implications of this in the section about the data files below)
- Participants were asked to wear the watch on the same wrist each day

### Motion Sensor Data

The watch collected sensor data from the WatchOS [CoreMotion](https://developer.apple.com/documentation/coremotion) framework. This data was collected at 10Hz for most of the participants, though for some it was collected at 50Hz. It was only collected when the watch was off its charger (and sometimes for a brief time right after being placed on the charger).

To provide consistent sensor rates and address any timestamp drift in the CoreMotion framework, data was resampled to a uniform 10Hz. (For details, see [the README](https://github.com/WSU-CASAS/Mobile-AL-Preprocessing-Tools) for the script used.)

Motion sensors include:
  - yaw
  - pitch
  - roll
  - rotation rate (x, y, z)
  - user acceleration (x, y, z) - user-induced (see [documentation](https://developer.apple.com/documentation/coremotion/cmdevicemotion/1616149-useracceleration))

### Location Data

In addition to motion sensors, data was also collected from the watch GPS using the [CoreLocation](https://developer.apple.com/documentation/corelocation) framework. The GPS location was queried every 5 minutes to save power.

Raw location values are not provided to preserve participant anonymity. These values are replaced by features derived from the raw location. The location type feature indicates the type of location that is being visited, as indicated by reverse-geocoding using [Nominatim](https://nominatim.org/). Additionally, each participant's mean location is calculated over all locations for that participant. Features are then generated reflecting the current location's distance from the mean in terms of latitude, longitude, and altitude.

Location sensors/features include:
  - location type
  - latitude distance from participant's mean
  - longitude distance from participant's mean
  - altitude distance from participant's mean
  - course (compass direction watch is moving)
  - speed (at which the watch is moving)
  - horizontal accuracy (uncertainty of location)
  - vertical accuracy (uncertainty of altitude)

### Battery Data

The state of the watch's battery was also collected. The rate of collection varied depending on battery usage, but is approximately every 5 minutes.

The battery state is one of the following:
  - `unplugged` - The watch is off its charger
  - `charging` - The watch is on the charger and is not full
  - `full` - The watch is on the charger and completely charged

While all three of these states were seen by the watch, the included data files almost exclusively include `unplugged` states due to only including times with valid motion data. (See more on this in the data file section below.)

### Activity Labels

Participants were periodically asked to respond to prompts on the watch. One of these prompts included a query for their current activity, which they chose from a preset list. (Note that the query frequency and activity choices varied among different groups of participants.)

These activities cover various activities of daily living (cooking, working, hygiene, etc). The label and the timestamp when the participant entered it are recorded. These labels are then present in the included data files (see notes on this in the next section).

## Included Data Files

Data from the participants are in the included CSV files, one file for each participant.

Each file has two header rows:
 1. The data field names
 2. The type of each data field:
    - `dt` - datetime
    - `f` - a numeric floating-point value
    - `s` - a string value

All following rows in the file represent the latest value of all sensors at the given timestamp, one timestamp per line. The timestamp corresponds to the timestamp of a resampled motion sensor event (as described in the Motion Sensor Data section above). *The values of all non-motion-sensor fields are the most recent value received at or before the given timestamp.*

This has some important implications:
  - Data is only included for timestamps when motion data was collected. As such, the data almost exclusively consists of times when the participant was wearing the watch. The time when the watch was charging is not included in these files, meaning there are gaps during each night (and other times) when the participant charged the watch.
  - The location sensor/feature values stay the same between location reading times (i.e. all rows between two readings have the same location values, those of the last location received). The value is updated on *the next motion timestamp after* the location reading occurred.
  - Similarly, battery state values stay the same between readings, and are updated on the next motion timestamp after a reading.
  - The user's activity label is provided on the next motion timestamp after the user confirmed their input. This means there is only one labeled row for each time the user chose an activity. However, the activity was likely to have occurred for some time span before the label input, so it is suggested that any interpretation of the labels take this into account. (We have a script to facilitate this in [our GitHub repository](https://github.com/WSU-CASAS/Mobile-AL-Preprocessing-Tools).)
   

The data fields included in each file are:
  - timestamp (in local time)
  - yaw (radians)
  - pitch (radians)
  - roll (radians)
  - rotation rate x (radians/second)
  - rotation rate y (radians/second)
  - rotation rate z (radians/second)
  - user acceleration x (G's)
  - user acceleration y (G's)
  - user acceleration z (G's)
  - location type (*)
  - latitude distance from mean (degrees) (*)
  - longitude distance from mean (degrees) (*)
  - altitude distance from mean (meters) (*)
  - course (degrees, relative to north) (*)
  - speed (meters/second) (*)
  - horizontal accuracy (meters) (*)
  - vertical accuracy (meters) (*)
  - battery state (*)
  - user activity label (*)

Features ending with (*) may not be provided at each timestamp (e.g. before the first location was read, no user label, etc).

We have created [a Python library](https://github.com/WSU-CASAS/Mobile-AL-Data) to assist in reading and writing these files on GitHub. Please see the README of that repository for details and usage examples.

### Files:
The participant data files are grouped into 3 groups based on the data collection conditions for each group. (Participants within the same group had similiar participant characteristics and data collection timeframes, prompt frequency, etc.)

The files are available [on the CASAS website](http://casas.wsu.edu/datasets/smartwatch/). Links to individual files are below:

  - [group1.zip](http://casas.wsu.edu/datasets/smartwatch/group1.zip) (9.1 GB), `sw1.p1` through `sw1.p15`
  - [group2.zip](http://casas.wsu.edu/datasets/smartwatch/group2.zip) (38.4 GB), `sw1.p16` through `sw1.p45`
  - [group3.zip](http://casas.wsu.edu/datasets/smartwatch/group3.zip) (2.2 GB), `sw1.p46` through `sw1.p49`
