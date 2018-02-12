# GT-2 thermistor experiment

**How good is this thermistor?**

## Experiment 1

Thanks to [David Crocker](https://github.com/dc42) who has identified my thermistor from a vague description I gave him, now I know what it was supposed to be like. It is a 104GT glass thermistor by ATC Semitec. The codename obviously means *"an (approximately) 100k&Omega; Glass Thermistor"*. This is how we know:

![measured data](SH-fit.1.png)

```
Steinthart-Hart coefficients, 3-point estimation:  A = 0.0008149464, B = 0.0002077971, C = 0.0000000996
Steinthart-Hart coefficients, NLS fit to data:     A = 0.0008235063, B = 0.0002046960, C = 0.0000001247
```

The colored lines are [resistance-temeperature tables](https://github.com/selkovjr/gt-2-thermistor-experiment/blob/master/gt-2-glass-thermistors.tab) from [ATC Semitec's datasheet](http://www.atcsemitec.co.uk/gt-2-glass-thermistors.html); black dots are the measurements I took from the thermistor I received with the [BiQu Diamond Hotend](https://www.biqu.equipment/products/diamond-3d-printer-extruder-reprap-hotend-3d-v6-heatsink-3-in-1-out-multi-nozzle-extruder-prusa-i3-kit-for-1-75-0-4mm).

While the data points from the first experiment appear to straddle the nominal curve for 104GT pretty nicely, the Steinhart-Hart model residuals reveal at least one flaw in this experiment: oscilations and possibly drift in the thermostat.

To provide temperature signal to the thermostat while I measured the resistance of the proband thermistor, I used an auxiliary thermistor tucked under the insulation blanket on top of the nozzle, next to the heater cartridge. While this set-up helped reduce the lag between the two thermistors, it was also insecure and probably accounted for much of the observed drift. Also, I neglected to tune the thermostat and it oscillated more than it normally does, making it difficult to track the set value. The two thermistors are not entirely dissimilar, so I thought the thermostat would work well if I simply swapped the auxiliary one in without even calibrating it. It worked, but not too well.


## Experiment 2

In the following experiment, instead of taking many measurements around each temperature setting in the hope that they would average close to it, I waited until the thermostat settled before taking each measurment. Also, unlike the first time, I made efforts to calibrate the auxiliary thermistor (using the Beta model) and tuned the thermostat to it. This time, I placed the auxiliary thermistor inside a screw hole in the nozzle, so that it was completely embedded in the metal, but still close to the surface (I was unable to screw it in deeper).

![hotend picture](diamond-hotend.png)

This arrangement resulted in a greater lag between the heater and the auxiliary thermistor. Due to intense radiation from nozzle surface, it also allowed for a greater (although unknown in either experiment) thermal gradient between the heater and the auxiliary thermistor, resulting in severe over-regulation. The thermostat overshot its setpoints by more than 10&deg;C and required more than 10 minutes to settle. But on the upside, the oscillations were well-damped and the new location of the auxiliary thermistor was mechanically and thermally stable, making measurements easily reproducible to within instrument precision.

The resulting model residuals are rather more tame than in the first exepriment:

![measured data](SH-fit.2.png)

```
Steinthart-Hart coefficients, 3-point estimation:  A = 0.0008336840, B = 0.0001991579, C = 0.0000001516
Steinthart-Hart coefficients, NLS fit to data:     A = 0.0008324170, B = 0.0001985115, C = 0.0000001625
```

With oscillations and drift subdued, this experiment reveals what appears to be an irreducible non-linearity of model error, which can now be recognized in the residuals of the first experiment.

What is it? Does it mean that the Steinhart-Hart model is inadequate? Can this pattern of deviation be caused by gradient-induced bias between the thermistor and the reference thermocouple? Imperfect thermocouple calibration?

## A thought experiment

*No hardware was harmed in the making of this observation*

The squiggly Steinhart-Hart residuals seen in Experiment 2 demand explanation. Steinhart and Hart have good reputations and thermistors are not known for exceedingly complex behavior. My thermocouple instrument has been recently calibrated by a standards lab, and I used boiling water and an ice bath to check that it was still sane before I set out to do these experiments. That makes the thermal gradient between the thermocouple and the proband thermistor a prime suspect. How big a gradient is it and is it possible that the shape of the field surrounding the probe and the thermistor is responsible for the warped resuduals?

To test that possibility, I built this thermal model of the nozzle using [a version of Energy2D by AnaMarkH](https://github.com/AnaMarkH/energy2d):

[diamond-nozzle.e2d](diamond-nozzle.e2d)
![measured data](diamond-hotend-200C.png)

> [The master build of Energy2D](http://energy.concord.org/energy2d/) did not work at this scale because of its grid size and resolution limitations. Also, AnaMarkH's version has an improved solver that eliminates a couple nasty artifacts. I did not have to build his version because the repo includes a pre-built jar; I just ran `java -jar energy2d/exe/energy2d.jar`.

This model is dodgy in too many ways to mention, yet it appears to be capable of simulating some properties of the field that could be responsible for the warp. I have no idea what those properties are.

Varying the temperature of simulated heater to match the observations at the probe produced the following dependence of thermistor-probe offset on probe temperature:

![thermal gradient](gradient.png)

This model is likely wrong about the magnitude of the field and even its shape, but somehow, transforming the data by adding a linear function of the simulated offsets (negating and scaling them **by a factor of 23**)  minimizes the Stheinhart-Hart resuduals, completely eliminating the squiggle:

![adjusted model!](SH-fit.corrected.png)

```
Steinthart-Hart coefficients, 3-point estimation:   A = 0.0006492344, B = 0.0002279751, C = 0.0000000558
Steinthart-Hart coefficients, NLS fit to data:     A = 0.0006509953, B = 0.0002275946, C = 0.0000000577
```

That is a remarkably good fit, free of obvious artifacts. The thermal model must be right about something. Now the question is, will it be right every time, or was this a stroke of luck?


## Sanity check

In an attempt to make the model a little more realistic, I added a heatsink:

[diamond-nozzle+heatsink.e2d](diamond-nozzle.heatsink.e2d)
![measured data](diamond-hotend+heatsink-200C.png)

Intuitively, it seems to be a better model. The heatsink flattened the field and inverted temperature difference between the thermistor and the probe. The maximum difference between all sampling points is now 4&deg;C. It was 7&deg;C in the previous model &mdash; an uncomfortably large amount.
