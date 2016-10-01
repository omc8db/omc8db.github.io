---
title: "FT-Raman Spectrometer (Ongoing)"
excerpt: "Signal processing algorithm development for next-generation on rover spectroscopy"
year: Ongoing
category: 'Mars Rover Design Team'
header:
  image: ftspec_render.png
  teaser: ftspec_render.png
sidebar:
  - title: "Role"
    text: "Signal Processing"
  - title: "Responsibilities"
    text: "Correct for vibration effects"
  - title: "Collaborators"
    text: "Frank Marshall, Katie Brinkner, John Maruska"
---

This year the rover team is building an FT-Raman Spectrometer. The monochometer used in previous years had no moving parts and projected the spectra onto a linear CCD. The sensitivity of the monochometer was limited by the resolution of the CCD and the size of its pixels.

An FT-Raman Spectrometer operates on a different principle. It has one moving part - a slide with a mirror. As the mirror moves along the slide, the optical setup "selects" a different frequency to project onto the single sensor. This lets us use a much larger and higher sensitivity sensor. With perfectly smooth slide movement, the data on the sensor will theoretically sweep through all of the observed wavelengths. A plot of measurement time vs intensity should be identical to a measurement of frequency vs

The problem with this design is that it assumes perfectly smooth slide movement. The wavelengths of light we care about (mostly visible) correspond to less than 1 micron of travel across the slide. Any imperfections in the slide or vibration could result in an unintelligible signal. We can't reach this precision mechanically within our budget, but we *can* provide a calibration signal that bounces off the same mirror and is sampled on another sensor simultaneously with the unknown sample spectra.

The calibration signal has a known value as a function of slide position. By finding a transformation that maps the vibration-scrambled calibration signal to an ideal calibration signal, it's possible to reconstruct where the slide was when a sample was taken. Applying the inverse of this transformation to the scrambled spectra measurement can 

That's all really wordy and abstract, so let's make it clearer with pictures.

## Goal:
Correct for imperfections in the slide and environmental vibrations in software after data collection.

## Tools used: 
Matlab

## Simulating the problem

In theory, this is what we should get on our sensors with a perfectly smooth slide movement:

![ideal](/images/ftspec_1.png)

The first step in simulating the problem was to produce irregular slide movement. I did this by lowpass filtering a stairstep function (to simulate jerky stepper motor motion) and adding random gaussian noise.

![motion](/images/ftspec_2.png)

Then, we resample the ideal signal with this new set of sampling points:

![scrambled](/images/ftspec_3.png)

Yuck! You can't tell anything about the spectra by looking at that.

## Theoretical Best Reconstruction

To make sure that the resampling didn't lose too much data to get anything, I plotted the resampled data using the simulated shaky movement as an index.

![theoretical best](/images/ftspec_4.png)

Yes! It should be possible to get something close to the original out of this signal.


## Practical Reconstruction

The theoretical best reconstruction assumes you know the exact position of the slide at all times. In reality, all we get is the calibration signal. Now there's a new problem: Aliasing.

Because the calibration signal goes up and down, there are multiple possible sample points that could produce a measurement at a certain intensity level. We want to pick one that's close to the last point we examined, and we want to ensure that it looks like the slide is advancing on average. To do this, I specify a maximum forward and backward amount to search and bias it to search forward for potential matches before searching backward.

![practical reconstruction](/images/ftspec_5.png)

This isn't identical to the theoretical best signal, but it preserves the location of four of the six major peaks well enough to be useful. 

## Future Work

Once the hardware prototype is up and running, I can find out how significant the vibration effects are in practice. The simulation represents a worst case extreme amount of vibration and noise; hopefully the real hardware is more tame.

[Source Code on Github](https://github.com/MST-MRDT/Science-Analysis){: .btn .btn--success .btn--large}