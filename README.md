# SoundFont Spec Implementation Test v3.0
A specification-compliance test for SoundFont synthesizers.

*Updated 1/20/25 by S. Christian Collins*

_**HELP WANTED!** If someone could send me audio of this test recorded on a Sound Blaster Live! with reverb and chorus enabled, I would be much obliged. You can upload your recording to [this bug report](https://github.com/mrbumpy409/SoundFont-Spec-Test/issues/2)._

## Overview

The following tests will check to see if your SoundFont player is compatible with the SoundFont [2.01](https://github.com/davy7125/soundfont-standard-v3/blob/master/sfspec21.pdf) and/or [2.04](https://github.com/davy7125/soundfont-standard-v3/blob/master/sfspec24.pdf) specification, using the hardware implementation found in the Sound Blaster Audigy 2 ZS as a reference point. This test will not cover every possible implementation flaw, but should give a decent overview of a SoundFont player's compatibility with the specification.

To run the test, load the accompanying SoundFont bank into your SoundFont synth as bank 0, and then play the MIDI file. Use `Audigy2 ZS, custom fx.ogg` in the `recordings` folder as a reference for how these tests should sound. Note that for some tests, FluidSynth provides a more ideal rendering, especially when it comes to the filter behavior.

## Tests

### Test #1: Volume envelope

This test uses all six envelope stages, each lasting exactly 1 second. The stages are delay, attack, hold, decay, sustain, and release. The attack should be a convex curve, while the remaining envelope stages should all be linear. Note that viewing these stages on a waveform view in dB, the attack will appear linear, and the other phases will appear concave. The test starts with a click. You should hear the note's volume change according to the following envelope shape:

            ____
           /    \
          /      \____
         /            \
    ____/              \

### Test #2: Modulation envelope

This test uses all six envelope stages, each lasting exactly 1 second. The stages are delay, attack, hold, decay, sustain, and release. The attack should be a convex curve, while the remaining envelope stages should all be linear. You should hear the note's pitch change according to the following envelope shape:

            ____
           /    \
          /      \____
         /            \
    ____/              \

### Test #3: Key number to decay

You should hear five tones at the same pitch, each one increasing in duration of the decay phase.

### Test #4: Key number to hold

You should hear five tones at the same pitch, each one increasing in duration of the hold phase.

### Test #5: Modulation LFO

You should hear the volume oscillating at a moderate speed (4 Hz). The LFO depth value is set to 6 dB, which means the sample volume should oscillate between +6 dB and -6 dB.

* **Test A:** Velocity = 127. Since there is no attenuation to the sound at instrument or preset level, the sound cannot be amplified. Therefore, the LFO will only cause a reduction in volume, but never a boost. In a waveform view, the volume peaks will appear to be "chopped off". See the notes for [test 12](#test-12-negative-attenuation-amount) below.

* **Test B:** Velocity = 90. Since there is attenuation to the sound due to the lower note velocity, the LFO can both amplify and attenuate the sound. In a waveform view, the volume oscillation should appear as a triangle wave.

### Test #6: Vibrato LFO

You should hear the pitch oscillating at a moderate speed (4 Hz).

### Test #7: Mod wheel to LFO

Which LFO is activated by the mod wheel (CC1)? According to the SoundFont 2.01 and 2.04 specs, the mod wheel should add 50 cents to the vibrato LFO (LFO2), resulting in a moderate-speed vibrato (4 Hz) in this test.

SoundFont 2.0 and earlier devices such as the Sound Blaster Live! or AWE32 had the mod wheel activating the modulation LFO (LFO1) instead, which will result in a very fast vibrato (8.176 Hz) in this test.

### Test #8: Scale Tune / Root Key

You should hear three tones, all at the same pitch.

### Test #9: Initial filter cutoff

You should hear a white noise wave filtered using a low-pass filter at each of the following values:

| Cutoff frequency (Hz) |
| ----------------------|
| 20                    |
| 50                    |
| 100                   |
| 200                   |
| 500                   |
| 1,000                 |
| 1,500                 |
| 2,000                 |
| 3,000                 |
| 4,000                 |
| 5,000                 |
| 6,000                 |
| 7,000                 |
| 8,000                 |
| 9,000                 |
| 10,000                |
| 12,000                |
| 15,000                |
| 17,500                |
| 20,000                |

Note that Sound Blaster hardware has an unusual filter that is fully open at 8,000 Hz. FluidSynth implements a proper 2-pole lowpass filter. [See my video here](https://www.youtube.com/watch?v=Qid15khj68k) on this topic.

### Test #10: Filter resonance

You should hear a white noise wave filtered using a low-pass filter at 4,000 Hz while increasing the filter resonance (Q) using the following values:

| Filter Q (dB) |
| ------------- |
| 0             |
| 5             |
| 10            |
| 15            |
| 20            |
| 25            |
| 30            |
| 35            |
| 40            |
| 50            |
| 70            |
| 96            |

The Sound Blaster hardware filter resonance appears to be capped to a maximum of 20 dB.

### Test #11: Attenuation amount

Each 1 dB of attenuation set at the instrument or preset level should only attenuate the sound by 0.4 dB. This is a quirk of the Sound Blaster hardware that should be emulated for compatibility with existing SoundFonts. Recent versions of Polyphone translate this value in the editor so you can see the actual attenuation amount.

For this test, you will hear a series of tones getting progressively quieter, based on the table below:

| dB in SoundFont | actual dB attenuated |
|:---------------:|:--------------------:|
| 0               | 0                    |
| 5               | 2                    |
| 10              | 4                    |
| 15              | 6                    |
| 20              | 8                    |
| 25              | 10                   |
| 30              | 12                   |

When measured, the volume decrease between each tone should be exactly 2 dB.

### Test #12: Negative attenuation amount

The same test as #11 is repeated, but with preset attenuation set to -12 dB. All tones should play at the same volume.

On Sound Blaster hardware, a playing sample (or "voice") can either be at full volume (0 dB) or attenuated below that level. No modulator, LFO or attenuation parameter value can raise the sample playback volume above this 0 dB limit. It is possible to set negative attenuation values at the preset level. However, if a voice has not been attenuated by a modulator (e.g., velocity, CC7, CC11), envelope, or by the attenuation value at the instrument level, then it cannot be boosted. In other words, a negative attenuation value in the preset can only boost the voice volume up to 0 dB, but never beyond.

### Test #13: Velocity to attenuation curve

Measured at the following velocities: 127, 111, 95, 79, 63, 47, 31, 15.

* **Test A:** default (should be 96 dB concave. The difference between 127 & 111 should be 2.34 dB)
* **Test B:** 144 dB concave
* **Test C:** 48 dB concave
* **Test D:** 96 dB linear
* **Test E:** modulator deleted from the instrument (all tones should be the same volume)

### Test #14: Velocity to initial filter cutoff curve

All tones should be the same volume, assuming test #13 is passed. Measured at the following velocities: 127, 111, 95, 79, 63, 47, 31, 15, with filter cutoff set to 4,000 Hz, and filter resonance at 10 dB.

#### A: default modulator

The result of this test will vary depending on which spec version is implemented by the SoundFont synth:

* **2.01 spec:** should perceive moderate filtering (-2400 cent curve) as the velocity decreases, with a distinct jump when passing over velocity=64 (due to the secondary modulator source=switch).
* **2.04 spec:** should perceive moderate filtering (-2400 cent curve) as the velocity decreases
* **Earlier spec versions:** no filtering should occur

Because of this inconsistent behavior and the different way each version of this modulator must be canceled/overridden by the SoundFont designer, many SoundFont synths including FluidSynth and BASSMIDI choose not to implement this default modulator.

#### B: -7200 cent curve

The result should be a smooth progression over a wide frequency range for the low pass filter, with noticeably more filtering at low velocities than the default velocity-to-FC modulator.

#### C: old SoundFont 2.0 behavior (SB AWE32/64 and Live!)

On the Sound Blaster Live!, a volume envelope attack of 8 ms or greater enables a velocity-to-FC modulator that reduces the filter cutoff at lower velocities. The same is true on AWE32/64 with volume envelope attack of 7 ms or greater. This test attempts to see if this behavior is emulated. If **Test A** does not result in a progressively filtered sound but **Test C** does, then this old behavior is emulated. If **Test A** sounds filtered, then ignore the results of this test.

#### D: 2.01 spec default modulator deleted from instrument

Tests to see if the synth responds to the 2.01 spec method of canceling the default velocity-to-FC modulator. If so, all tones should have the same filtering. Ignore this test if the default modulator is not implemented (see **Test A**).

#### E: 2.04 spec default modulator deleted from instrument

Tests to see if the synth responds to the 2.04 spec method of canceling the default velocity-to-FC modulator. If so, all tones should have the same filtering. Ignore this test if the default modulator is not implemented (see **Test A**).

### Test #15: CC1 to filter cutoff

You should hear a smooth opening and closing of the filter, which is being controlled by the mod wheel (CC1). If this isn't implemented, you will probably hear vibrato instead.

### Test #16: Sample Offset

You will hear either "supported" or "not supported". Strangely, the Sound Blaster Live! does not support sample offset, even though the AWE32/64 does.

### Test #17: Reverb

* **Test A:** Instrument-level reverb = 0%, 33.3%, 66.6% and 100%.
* **Test B:** MIDI CC 91 reverb = 0% (0), 33.3% (42), 66.6% (85) and 100% (127).
* **Test C:** Both instrument-level and MIDI CC91 reverb = 0%, 33.3%, 66.6% and 100%.

### Test #18: Chorus

* **Test A:** Instrument-level chorus = 0%, 33.3%, 66.6% and 100%.
* **Test B:** MIDI CC 93 chorus = 0% (0), 33.3% (42), 66.6% (85) and 100% (127).
* **Test C:** Both instrument-level and MIDI CC93 chorus = 0%, 33.3%, 66.6% and 100%.

### Test #19: Sample interpolation quality

This test must be performed by ear. If the sound is grainy, a low quality interpolation method (e.g., "linear") is probably being used.

### Test #20: Pitch bend

* **Test A:** Default (+/- 2 semitones) --you will hear middle C bend down a whole step and back, then up a whole step and back.
* **Test B:** Modulator removed (should hear no pitch change)
* **Test C:** Inverted --you will hear middle C bend up a whole step and back, then down a whole step and back.

### Test #21: Exclusive class

Very short noise = exclusive class is supported
Long noise = exclusive class is not supported
