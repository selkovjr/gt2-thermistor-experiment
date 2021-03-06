# GT-2 thermistor experiment

**How good is this thermistor?**

## Experiment 1

Thanks to [David Crocker](https://github.com/dc42) who has identified my thermistor from a vague description I gave him, now I know what it was supposed to be like. It is a 104GT glass thermistor by ATC Semitec. The codename obviously means *&ldquo;an (approximately) 100k&Omega; Glass Thermistor&rdquo;*. This is how we know:

![measured data](SH-fit.1.png)

```
Steinthart-Hart coefficients, 3-point estimation:  A = 0.0005949118, B = 0.0002426185, C = -0.0000000180
Steinthart-Hart coefficients, NLS fit to data:     A = 0.0008235064, B = 0.0002046959, C = 0.0000001247
```

The colored lines are [resistance-temperature tables](https://github.com/selkovjr/gt-2-thermistor-experiment/blob/master/gt-2-glass-thermistors.tab) from [ATC Semitec data sheet](http://www.atcsemitec.co.uk/gt-2-glass-thermistors.html); black dots are the measurements I took from the thermistor I received with the [BiQu Diamond Hotend](https://www.biqu.equipment/products/diamond-3d-printer-extruder-reprap-hotend-3d-v6-heatsink-3-in-1-out-multi-nozzle-extruder-prusa-i3-kit-for-1-75-0-4mm).

While the data points from the first experiment appear to straddle the nominal curve for 104GT pretty nicely, the Steinhart-Hart model residuals reveal at least one flaw in this experiment: oscillations and possibly drift in the thermostat.

To provide temperature signal to the thermostat while I measured the resistance of the proband thermistor, I used an auxiliary thermistor tucked under the insulation blanket on top of the nozzle, next to the heater cartridge. While this set-up helped reduce the lag between the two thermistors, it was also insecure and probably accounted for much of the observed drift. Also, I neglected to tune the thermostat and it oscillated more than it normally does, making it difficult to track the set value. The two thermistors are not entirely dissimilar, so I thought the thermostat would work well if I simply swapped the auxiliary one in without even calibrating it. It worked, but not too well.

> It did not work at all for the 3-point method, with the first and last temperature points picked, along with one in the middle. The C coefficient can't be negative because the inverse Steinhart-Hart equation depends on a square root of it. The 3-point results varied wildly in this experiment &mdash; from nonsensical to plausible, depending on the choice of points. Sergei Severin once advised his then-young student Schnoll, who lamented his inability to achieve convergent measurements of reaction rates: &ldquo;Take fewer measurements, and your life will be easier&rdquo;. The rest of this little report describes what happens when one doesn't heed the wisdom of the greats.

## Experiment 2

In the following experiment, instead of taking many measurements around each set point in the hope that they would average close to it, I waited until the thermostat settled before taking each measurement. Also, unlike the first time, I made efforts to calibrate the auxiliary thermistor (using the Beta model) and tuned the thermostat to it. This time, I placed the auxiliary thermistor inside a screw hole in the nozzle, so that it was completely embedded in the metal, but still close to the surface (I was unable to screw it in deeper).

![hotend picture](diamond-hotend.png)

This arrangement resulted in a greater lag between the heater and the auxiliary thermistor. Due to intense radiation from nozzle surface, it also allowed for a greater (although unknown in either experiment) thermal gradient between the heater and the auxiliary thermistor, resulting in severe over-regulation. The thermostat overshot its setpoints by more than 10&deg;C and required more than 10 minutes to settle. But on the upside, the oscillations were well-damped and the new location of the auxiliary thermistor was mechanically and thermally stable, making measurements easily reproducible to within instrument precision.

The resulting model residuals are rather more tame than in the first experiment:

![measured data](SH-fit.2.png)

```
Steinthart-Hart coefficients, 3-point estimation:  A = 0.0008336840, B = 0.0001991579, C = 0.0000001516
Steinthart-Hart coefficients, NLS fit to data:     A = 0.0008324170, B = 0.0001985115, C = 0.0000001625
```

With oscillations and drift subdued, this experiment reveals what appears to be an irreducible non-linearity of model error, which can now be recognized in the residuals of the first experiment.

What is it? Does it mean that the Steinhart-Hart model is inadequate? Can this pattern of deviation be caused by gradient-induced bias between the thermistor and the reference thermocouple? Imperfect thermocouple calibration?

## A thought experiment

*No hardware was harmed in the making of this observation*

The squiggly Steinhart-Hart residuals seen in Experiment 2 demand explanation. Steinhart and Hart have good reputations and thermistors are not known for exceedingly complex behavior. My thermocouple instrument has been recently calibrated by a standards lab, and I used boiling water and an ice bath to check that it was still sane before I set out to do these experiments. That makes the thermal gradient between the thermocouple and the proband thermistor a prime suspect. How big a gradient is it and is it possible that the shape of the field surrounding the probe and the thermistor is responsible for the warped residuals?

To test that possibility, I built this thermal model of the nozzle using [a version of Energy2D by AnaMarkH](https://github.com/AnaMarkH/energy2d):

[diamond-nozzle.e2d](diamond-nozzle.e2d)
![measured data](diamond-hotend-200C.png)

> [The master build of Energy2D](http://energy.concord.org/energy2d/) did not work at this scale because of its grid size and resolution limitations. Also, AnaMarkH's version has an improved solver that eliminates a couple nasty artifacts. I did not have to build his version because the repo includes a pre-built jar; I just ran `java -jar energy2d/exe/energy2d.jar`.

This model is dodgy in too many ways to mention, yet it appears to be capable of simulating some properties of the field that could be responsible for the warp. I have no idea what those properties are.

Varying the temperature of simulated heater to match the observations at the probe produced the following dependence of thermistor-probe offset on probe temperature:

![thermal gradient](gradient.png)

This model is likely wrong about the magnitude of the field and even its shape, but somehow, transforming the data by adding a linear function of the simulated offsets (negating and scaling them *by a factor of 23*)  minimizes the Steinhart-Hart residuals, completely eliminating the squiggle:

![adjusted model!](SH-fit.corrected.png)

```
Steinthart-Hart coefficients, 3-point estimation:  A = 0.0006492344, B = 0.0002279751, C = 0.0000000558
Steinthart-Hart coefficients, NLS fit to data:     A = 0.0006509953, B = 0.0002275946, C = 0.0000000577
```

That is a remarkably good fit, free of obvious artifacts. The thermal model must be right about something. The question is, will it be right every time, or was this a stroke of luck?


## Sanity check

In an attempt to make the model a little more realistic, I added a heatsink:

[diamond-nozzle+heatsink.e2d](diamond-nozzle.heatsink.e2d)
![measured data](diamond-hotend+heatsink-200C.png)

Intuitively, it seems to be a better model. The addition of the heatsink equalized the field and flipped the gradient between the thermistor and the probe. The spread between all sampling points is now 4&deg;C. In the previous model, it was 7&deg;C &mdash; uncomfortably large. Also, the difference between the thermistor and the tip of the nozzle (this project's deliverable) seems more reasonable.

Interestingly, while this model is a stronger hint at the possibility that the thermal design of the Diamond Hotend was guided by rational thought, it does nothing to improve the warped Steinhart-Hart residuals. The only sensible transformation that makes it a minimizer of residuals is multiplication by zero.

![thermal gradient](gradient.heatsink.png)

> The series labeled *fool's errand* (red) shows how the gradient in the heatsink model varies with probe temperature. It is flipped and scaled 15% for easier comparison with the previous model, *stroke of luck* (green).

This observation begs the question of how many non-trivial model configurations are possible that both match the observed temperature dependence at the probe and properly minimize Steinhart-Hart residuals for the thermistor. Another question (and probably one that should have been answered first) is whether the minimization of residuals by a model-derived transformation of probe temperatures makes it a good proxy.

The only answer obtained so far is that the warping of Steinhart-Hart residuals as a result of calibration by proxy can possibly be caused by temperature gradient.

## Closed-loop testing

All observations seem to indicate that a substantial thermal resistance exists between the thermistor site and the more peripheral location of the thermocouple probe. Therefore, no calibration attempt involving surfac contact between the hotend and a probe, or even inserting the probe into existing holes, will ever work in a live printer set-up, which, by design, is subject to high temperature gradients (what with that fan kicking in at 45&deg;C and going full speed at 150&deg;C). The only situations that allow accurate calibration are those that minimize the gradient &mdash; either by reducing the distance between the thermistor and the probe or by insulating the hotend. Neither approach is practical without dismantling the hotend. The best way to do it is to take the thermistor out to calibrate it in a bath thermostat. At the high end of the range, it may need to be a molten metal bath.

In this last experiment, letting the printer take control of the hotend temperature with the proband thermistor in the loop, I observed numeric differences between set-point temperatures and probe readings. The following graph shows these differences plotted for each conceivable set of Steinhart-Hart coefficients.

![thermal gradient](proxy.png)

> The noticeable kinks in the curves, especially in the blue gradient-compensated ones, correspond to fan threshold temperatures: on at 20% power at 45&deg;C; full power at 150&deg;C. Without the fan, the curves would be rather less steep, but then everything around the hotend would melt.
>
> The weird behavior at room temperature is probably caused by measurement error, at least partially, and it as also consistent with the possibility that the series resistor on the board is not exactly 4700&Omega; and has non-zero temperature dependence. I will update these observations when I find it and measure its actual resistantce.

### Nominal values (red curve)

Setting thermostat parameters from ATC Semitec data sheet resulted in a limited but uncomfortably large deviation. Especially uncomfortable is its direction: it makes the thermistor appear cooler than the probe. Still, in the absence of calibration data, it is not a terrible solution. With proper tune-up, it can result in a working thermostat. The downside of this approach is that it is impossible to tell how accurate it is.

### Nominal values with measured room-temperature resistance (faint red curve)

Measuring thermistor resistance at room temperature can be done fairly accurately. With ambient temperature at 21&deg;C, the gradient inside the hotend is negligible. Adjusting the Steinhart-Hart equation using the measured room-temperature resistance (98400&Omega; in this case) resulted in somewhat better behavior. It is still wrong, although apparently not as wrong as with all nominal values informing the thermostat.

Note that the measured resistance is within the factory tolerance of 3%.

### Zero gradient assumption (cyan curve)

A na&iuml;ve approach to calibration would be to ignore thermal resistance between the thermistor and the probe. This may be a fair assumption in the case of a small heater block, such as the original RepRap block or pretty much every hotend seen on the market today. It gets even better if the heater block is insulated. I noticed the current trend in 3D printer design is to insulate the hotend; that should improve temperature accuracy, possibly allowing for *in situ* calibration.

In the Diamond Hotend, the observed temperature dependence of probe deviation completely invalidates the assumption of zero gradient. It is, indeed, the least deviant of all models tested, but it is such by design. It achieves this result by bending the truth, both figuratively and literally. I find no comfort in a (relatively) small deviation knowing that it could only be achieved with a non-convex transformation of temperature-resistance dependence. It implies a non-convex (and negative) gradient &mdash; a certain impossibility.

It is still likely the least wrong of all wrong solutions; probably good enough to set up a well-working thermostat (I tested that extensively), but not good enough for accurate temperature measurement.

### Well-behaved solution based on gradient compensation (blue curves)

Notwithstanding the murky machinations that lead me to discover a plausible gradient compensation, the model that takes it into account seems to make the most sense. Here is the list of reasons why I like it.

* It does not produce non-convex behavior.
* It minimizes Steinhart-Hart residuals in a sane way. The residuals appear to be random, symmetric, and are commensurate with instrument error (cold junction instability, reference voltage accuracy, and rounding errors in digital displays). This result and the non-convexity of probe deviation may be aspects of the same phenomenon; if they are, it is still nice to be a able to observe it by a couple different methods.
* This solution does not imply a heat sink inside the nozzle or a heat source on the outside. The observed steady-state behavior always indicates a hotter core.
* The dynamic behavior during heat-up and cool-down (not presented here due to difficulties in simultaneous recording) shows a quasi-symmetric probe lag. The system exhibits higher probe deviation during heat-up and lower or negative deviation during cool-down. The inversion of the gradient is possible because the tips of the heatsinks are closer to the thermistor than to the probe. In fact, the hole into which I inserted the probe is equidistant and most removed from the nearest pair of heatsinks; the smooth surface of the nozzle in a natural convective flow is less efficient than the fan-cooled heatsinks, thus  *momentary* inversion of the gradient during cool-down makes more sense than if it were constantly positive, let alone constantly negative. I surmise that if my thermistor model was off by a large amount, it would not be sensitive to such short-lived inversions.
* A circumstantial, yet reassuring, evidence of good behavior is found in the fact that the Steinhart-Hart coefficients obtained with gradient compensation resulted in the most agile thermostat response following a tune-up (about 20% better than the next best result).
* An imprecise but also reassuring indication of success is the behavior of several different materials in the nozzle. I tested it with two different brands of PLA, an ABS, a PETG, a TPU, and a Nylon. All flowed well within the recommended range of printing temperatures. In particular, TPU has a sharp optimum and goes through a series of transitions in a relatively short temperature range. The indicated range for the filament I have is 225..235&deg;C. At 195&deg;C, it is just about possible to extrude it under considerable pressure; at 200&deg;C it expands 2x to 3x on leaving the nozzle; at 210&deg;C, it begins to flow with only minimal expansion; at 220&deg;C, it extrudes easily under very little pressure; at 230&deg;C, it drips and completely drains the cavity within a couple minutes; it remains fluid and clear up until about 240&deg;C, at which point it boils and smokes, sputterig drops of foam.

#### Why two curves?

As I was approaching the end of the first &lsquo;well-behaved&rsquo; series (the dark blue curve), the print head shifted along the X-axis and the probe fell off. Since I was unable to recall the original X-position, I left the print head where it was and re-inserted the probe. Having made a few more measurements in the new position, I discovered that I was no longer on the same curve and decided to re-run the entire series. The resulting curve is plotted in light blue.

The teachable moment here is that the accuracy of the probe (and likewise, possibly, of the proband thermistor closer to the core) depends on the way it is inserted, suggesting a strong small-scale gradient around it (to the tune of several &deg;C per mm). Also, very likely, it depends on natural convective flow around the hotend, which in turn depends on print head position. We already know from thermal modeling that the interaction between the air and the field within the nozzle is not negligible. So the fact that this perturbed experiment resulted in a different curve of the same character can also be taken as an indication of good behavior, as well as of the possibility of irreducible error due to the changing environment.
