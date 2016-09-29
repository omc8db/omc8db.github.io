---
title: "Rover Signals Overhaul (2016)"
excerpt: "Maintaining Telemetry and Control in RF Hostile Situations"
category: 'Mars Rover Design Team'
header:
  image: zenith_signals.jpg
  teaser: zenith_signals.jpg
sidebar:
  - title: "Collaborators"
    text: "Judah Schad, Josh Reed, Gbenga Osibodu"
---

Wi-Fi wasn't going to cut it any more.

The University Rover Challenge is a very harsh radio environment. Competitors need to control their rovers through sand dunes and box canyon walls over multiple kilometers. From 2014-2015, the Missouri S&T Mars Rover Design Team experienced multiple drop outs, signal loss incidents and out of control scenarios. In 2016 they experienced none.

# Radio Failures

Horizon (the 2015 rover) used 2.4 Ghz Ubiquiti Bullet radios with omnidirectional quarter wave antennas. It was basically a wi-fi router with 600mW output power. These worked very well in testing, but had problems in the field. 

* Multipathing: We noticed our signals dropping out when we went into canyons or depressions, even with clear line of sight. The radios would report having a strong signal, but no data would go through.
* Loss of line of sight: Whenever we lost sight of the rover, the signal dropped
* Interference: Other teams were using 2.4 Ghz, and their signals were interefering with ours


# Reproducing the failure

The first thing we needed to do was reproduce the situation that caused Horizon's radio systems to fail. We couldn't do it in the design center shop, so we had to go out to the field.

![Fugitive Beach](images/fugitive_beach_1.jpg)

Fugitive Beach was an abandoned quarry near Rolla. A few years ago, it was converted into a swimming pool. It's history as a quarry left it with lots of exposed rock faces. I went out with my cell phone and marked some GPS points of interest, then we took the rover out to test our existing signal system

![Testing the old rover systems](images/fugitive_beach_2.jpg)


# Evaluating Options

We chose two options as 

## AX.25 at 147 Mhz

From my time in the Amateur Radio club, I was familiar with the AX.25 protocol used in the Amateur Packet Reporting System (APRS). I regularly recieve AX.25 packets on the 2 Meter band from 30 miles away through obstructions, so it seemed like a good candidate. The next step was to build a prototype that could be used as an ethernet router.

For a quick prototype, I picked out a Raspberry Pi as my control computer and a Baofeng UV-5R as my radio. To connect them, I bought a USB sound card from Amazon. When I plugged it in, it immediately started transmitting.

Some searching later, I discovered the issue was that the audio ports on the UV-5R had separate grounds, and connecting the grounds was used to activate push-to-talk. I found a github project to build an adapter and ordered one from OshPark

![adapter](images/uv5r_adapter.jpg)

On the software side, Linux has built in AX.25 support. I found a frontend called soundmodem that allows configuring the AX.25 linux interface and configued it to use the USB soundcard.

There was one final piece - preventing the radios from transmitting all the time. I enabled an audio threshold on the radio to allow it to trigger when the 

## Commercial FSK transmitter at 433 Mhz

I also found a very inexpensive transciever pair on AliExpress. There was TODO



## Commercial OFDM system at 900 Mhz

The 900 Mhz Gorilla in the field was the Ubiquiti Rocket M900. This was basically the rocket, but operating at a lower frequency. Judah Schad identified this system and bought one to test.

We also got cloverleaf antennas.

# Results

AX.25 had sufficient range, but suffered from extremely limited throughput and latency. Because I was only using one simplex radio at each side, I had to use a channel sharing scheme and the half-duplex mode 

# Integration

Because the Ubiquiti Rocket M900 had the same PoE standard and routing characteristics as the bullet M2, electrical integration was just a matter of swapping them out. I zip tied the radio in while the team's mechanical engineers designed a mount. 

Antennas werea different issue. After talking with James Zandstra, mechanical lead, we settled on suspending.

picture: the AX.25 System

picture: testing