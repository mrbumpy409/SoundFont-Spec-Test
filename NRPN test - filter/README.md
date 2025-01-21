# Filter NRPN Test
A compliance test for AWE32 NRPN support in SoundFont synthesizers.

*Updated 1/20/25 by S. Christian Collins*

This will test the filter cutoff (FC) and resonance (Q) NRPNs introduced with the AWE32. All later Sound Blaster cards with an onboard SoundFont synthesizer support these NRPNs, to the best of my knowledge, though their behavior might differ somewhat from the original AWE32 implementation. This test requires a SoundFont synth compatible with the full 2.01/2.04 spec.

To run the test, load the SoundFont into bank 1 or 0 and play the accompanying MIDI file.

The following tests all use a white noise waveform, and tests \#1–3 have 24 dB filter Q applied within the preset. This emphasis is to make the filter cutoff point more visible when viewed in a spectrogram.

1. If the AWE32 FC NRPN is implemented, you should hear a filter sweep from 100 to 8000 Hz and then back down to 100 Hz.

2. You will hear a filter sweep that emulates the AWE32 FC NRPN filter curve via CC1 modulator. On my Audigy2, the result is identical to test \#1 above.

3. You should hear a sustained white noise wave while the filter switches every two seconds through the following values:
   * 0 (100 Hz)
   * 13 (157 Hz)
   * 25 (237 Hz)
   * 38 (371 Hz)
   * 51 (581 Hz)
   * 64 (910 Hz)
   * 76 (1377 Hz)
   * 89 (2156 Hz)
   * 102 (3376 Hz)
   * 114 (5108 Hz)
   * 127 (8000 Hz)
   
   Each jump in the filter should sound roughly like the interval of a fifth.

4. The same as the above test, but with the AWE32 FC NRPN filter curve emulated via CC1 modulator. On my Audigy2, the result is identical to test \#3 above.

5. You should hear a series of filter sweeps, each with progressively higher filter Q:
   * 0 (0 dB)
   * 13 (2.03 dB)
   * 25 (3.91 dB)
   * 38 (5.94 dB)
   * 51 (7.97 dB)
   * 64 (10 dB)
   * 76 (11.88 dB)
   * 89 (13.91 dB)
   * 102 (15.94 dB)
   * 114 (17.81 dB)
   * 127 (19.84 dB)

6. The same as the above test, but with the AWE32 filter Q NRPN emulated via CC2 modulator. On my Audigy2, the result is identical to test \#5 above.

## Emulating FC/Q NRPN Behavior with Modulators

The Audigy's filter NRPN implementation can be easily emulated using modulators, the parameters of which were determined by measuring the results of several tests. This is useful information for SoundFont synth designers, who might be able to leverage existing modulator code when implementing support for the FC/Q NRPNs. Note that the Audigy's implementation is not 100% identical to the original AWE32 implementation, particularly regarding the filter Q NRPN, which was based on a lookup table with 16 entries and resonance that would vary in strength depending on the filter cutoff.

To emulate the Audigy's filter cutoff NRPN for a voice:
1. Override the voice's filter cutoff, setting it to: **100 Hz**
2. Set the filter cutoff NRPN to modulate filter cutoff based on the following modulator:
   * Curve: **unipolar linear positive**
   * Amount: **7646** (timecents)
   * Destination: **filter cutoff**

To emulate the Audigy's filter Q NRPN for a voice:
1. Override the voice's filter Q, setting it to: **0 dB**
2. Set the filter Q NRPN to modulate filter Q based on the following modulator:
   * Curve: **unipolar linear positive**
   * Amount: **200** (20 dB)
   * Destination: **filter Q**
