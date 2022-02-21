+++
title = "Missouri S&T Mars Rover Radio Comms"
date = 2016-08-01

[taxonomies]
tags = ["embedded", "college", "rover"]
+++


The University Rover Challenge is a very harsh radio environment. Competitors need to control their rovers through sand dunes and box canyon walls over multiple kilometers. From 2014-2015, the Missouri S&T Mars Rover Design Team experienced multiple drop outs, signal loss incidents and out of control scenarios. In 2016 they experienced none.

# Radio Failures

Horizon (the 2015 rover) used 2.4 Ghz Ubiquiti Bullet radios with omnidirectional quarter wave antennas. It was basically a wi-fi router with 600mW output power. These worked very well in testing, but had problems in the field. 

* Multipathing: We noticed our signals dropping out when we went into canyons or depressions, even with clear line of sight. The radios would report having a strong signal, but no data would go through.
* Loss of line of sight: Whenever we lost sight of the rover, the signal dropped
* Interference: Other teams were using 2.4 Ghz, and their signals were interefering with ours


# Reproducing the failure

The first thing we needed to do was reproduce the situation that caused Horizon's radio systems to fail. We couldn't do it in the design center shop, so we had to go out to the field.

![Fugitive Beach](/images/fugitive_beach_1.jpg)

Fugitive Beach was an abandoned quarry near Rolla. A few years ago, it was converted into a swimming pool. It's history as a quarry left it with lots of exposed rock faces. I went out with my cell phone and marked some GPS points of interest, then we took the rover out to test our existing signal system.

Special points of interest: A box canyon - the worst case scenario for multipath, a recessed circular depression without line of sight to the base station, a ditch with no line of sight to the base station, and an area behind a large metal shipping crate.

The existing system worked as expected - good quality with direct line of sight, no throughput in the box canyon, no connection out of line of sight.

# Evaluating Options

## AX.25 at 147 Mhz

From my time in the Amateur Radio club, I was familiar with the AX.25 protocol used in the Automatic Packet Reporting System (APRS). I regularly recieve AX.25 packets on the 2 Meter band from 30 miles away through obstructions, so it seemed like a good candidate for long range rover control.

For a quick prototype, I picked out a Raspberry Pi as my control computer and a Baofeng UV-5R as my radio. To connect them, I bought a USB sound card from Amazon. When I connected everything, the radio immediately started transmitting without warning. The only thing that stopped it was unplugging one of the audio cables.

Some searching later, I discovered the issue was that the audio ports on the UV-5R had separate grounds, and connecting the grounds was used to activate push-to-talk. I found an [adapter design on github](https://github.com/johnboiles/BaofengUV5R-TRRS), ordered a board from OshPark, and assembled it. Problem solved!

On the software side, Linux has built in AX.25 support. I found a frontend called soundmodem that allows configuring the AX.25 linux interface and configued it to use the USB soundcard. A few lines of networkManager config bridged the AX.25 virtual network interface to the ethernet adapter on the pi. The result was a raspberry pi that looked to the other devices on the network just like the Bullet M2 it was replacing.

There was one final piece - ensuring that the transmit function on the radio could be controlled. I enabled an audio threshold on the radio to allow it to trigger when the computer transmitted. 

Once everything was working in the shop it was time for range testing. I left one unit in the shop and brought the other to the top of the student dorms on the other side of town.

![Roof Testing](/images/roof_testing.jpg)

Result: Success! DIY long range digital communication achieved.

## Commercial OFDM system at 900 Mhz

The 900 Mhz Gorilla in the field was the Ubiquiti Rocket M900. This was basically the bullet, but operating at a lower frequency. Judah Schad identified this system and bought one to test.

We also got cloverleaf antennas to test the effects of circular polarization. Judah theorized that it would cut down on multipath interference

# Test Results

![The Rover with test antennas mounted](/images/fugitive_beach_antennas.jpg)

AX.25 had more than sufficient range. The very restricted throughput wasn't actually a big issue because the control and telemetry packets required very little banwidth. It's biggest weakness was latency. Because I was only using one simplex radio at each side, I had to use a time slicing half-duplex mode to prevent conflicts on the shared channel. Once network utilization started to increase, the result was ugly:

![A twenty-five SECOND ping](/images/longping.jpg)

Although the Rocket M900 didn't have the extreme range, the throughput was on par with the wi-fi based system, far ahead of the AX.25 system. There was enough network capacity to run live video streams.

When we tried it with the circularly polarized cloverleaf antennas, the canyon test changed dramatically: The multipath issues went away completely. Other tests were not significantly affected.


**Winner: Rocket M900 with Cloverleaf Antennas**

# Integration

Because the Ubiquiti Rocket M900 had the same PoE standard and routing characteristics as the bullet M2, electrical integration was just a matter of swapping them out. I zip tied the radio in while the team's mechanical engineers designed a mount. 

Antennas werea different issue. After talking with James Zandstra, mechanical lead, we decided to use long carbon fiber rods to attach the antennas to the back of the rover.

![Final antennas setup](/images/final_rover_antennas.jpg) 