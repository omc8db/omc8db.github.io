---
title: "Autonomous Navigation (2017)"
excerpt: "Navigation through rough environments"
category: 'Mars Rover Design Team'
header:
  image: autonomy-large.jpg
  teaser: autonomy.jpg
sidebar:
  - title: "Role"
    text: "Subsytem Lead & Main Programmer"
  - title: "Result"
    text: "First place at URC2017, perfect score in the autonomy section"
  - title: "Collaborators"
    text: "Eddie Koharik"
  - title: "Successor"
    text: "Sarah Dlouhy"
---

[Source Code on Github](https://github.com/MST-MRDT/AutonomyBoard-Software){: .btn .btn--success .btn--large}\

There are four basic components to an autonomous navigation system:

1)  Decision making / goal setting

2)  Localization

3)  Path Planning

4)  Path Execution

Because it was our first year deploying any autonomous capability, I
decided to keep the system design as simple as possible. My biggest
concern was making sure I field tested everything extensively, and every
little bit of complexity I added would make testing more difficult. We
ended up with a minimalist, but very robust navigation package.

The software was implemented as a python program running on a Raspberry
Pi 2. It communicated over a dedicated Ethernet link with a UDP based
packet protocol to the microcontrollers that ran the sensors and
actuators.

Goal Setting
============

The rover's goals are defined as a list of coordinates. A marker may or
may not be in the presence of each coordinate.

A state machine manages the rover's actions.

Localization
============

GPS and a magnetometer are used to estimate the position of the rover.
Position is defined as a set of GPS coordinates (floating point, decimal
degrees) and a bearing (degrees clockwise from due north, floating
point).

Both the GPS and magnetometer are sources of error, so calibration and a
simplified sensor fusion system were used to produce a more accurate
estimate of the position of the rover.

We used a cheap GPS unit from sparkfun. I don't remember the exact
sensor, but it was about \$5 and updated at around 1 Hz with an
estimated accuracy of 5 meters. It wasn't a great GPS, but it was good
enough. Spending even a little more money could give lots of improvement
here.

We also used a cheap magnetometer. Magnetometer calibration was an
interesting subject to learn, and there turned out to be a lot more to
it than I first thought.

For the navigation system, accurate bearing information was much more
important than accurate location. +/- 5 meter accuracy at ranges of 100+
meters was totally fine, but a 10 degree error in heading information
was much more difficult. Unfortunately, that's what we got from cheap
magnetometers.

To get usable bearing, I implemented the simplest possible "sensor
fusion" between GPS and Magnetometer data, and used a cross-track
correction term in the Path Execution algorithm to compensate for
persistent errors. The "sensor fusion", if you could even call it that,
was just a weighted average between the bearing the magnetometer gave us
and the delta between the two most recent GPS points. The weighting gave
full trust to the magnetometer at low speeds, and gradually brought in
the GPS delta as speeds rose.

Path Planning
=============

Path planning was very basic: Go in a straight line until you get close
to a waypoint, then search with machine vision for a target marker. If
there is a target marker, go straight to it. Otherwise, continue to seek
the GPS waypoint.

Path Execution
==============

Path execution had multiple alternate algorithms, optimized for
different conditions the rover could encounter. Switching between these
algorithms was managed by the toplevel state machine.

The core algorithm in path execution was blind navigation. Blind
navigation was a traditional Proportional-Integral-Derivative (PID)
controller. This controller took in the result from the localization and
path planning stages, and produced a course correction from the
difference between them. To reduce the number of cases we had to test,
speed was held to be constant in this mode. Corrections were only
applied to heading.

One problem encountered during testing was that a miscalibrated
magnetometer, slippery sloped surfaces at odd angles, or even natural
sensor noise could cause the rover to take a spiraling path to a goal
instead of a straight line. To fix this, I added a second correction
term for cross track error to the correction produced by the PID
controller. Cross track error is the sideways deviation from the
straight line path between last waypoint and the current goal.
Crosstrack correction strength was a linear function of the magnitude of
the crosstrack error.

Once I found an appropriate coefficient for the crosstrack correction
strength, all of the spiraling path problems went away. Interestingly,
the rover became robust to much larger errors in localization than I had
anticipated -- even a 30 degree error in the initial magnetometer
calibration was quickly corrected.

The secondary path execution algorithm was a machine vision based target
seeker. Our targets were tennis balls on a stick. This was really
convenient for me, because one of the free OpenCV examples draws circles
around tennis balls. I took some footage while field testing and tweaked
the parameters in the example until it could reliably draw a circle
around a tennis ball in a desert environment.

Estimating the position of the tennis ball was a quick extension. The
diameter of the tennis ball on screen was inversely related to how far
away it was. I measured the field of view of our cameras by setting them
down on a protractor. The real life diameter of a tennis ball is
regulated by the International Tennis Federation, so I took their value
as a constant. Armed with this information, I was able to calculate the
distance to the tennis ball based on its size on screen. I estimated the
angle based on how far the center of the circle was from the middle of
the image horizontally.

A distance and angle estimate were all I needed to feed another PID
loop. The goal of this control loop was to point directly at the tennis
ball and put it exactly one meter away. I picked a distance constraint
of one meter so we wouldn't accidentally crash into the marker.

All of the PID loops ran at high priority, but without any realtime
guarantees. I wrote a dynamic timestep PID implementation so the system
could stay accurate even under wild variations in frequency. The control
loops ran at a minimum update frequency of 15 Hz, but more commonly
averaged around 200 Hz.
