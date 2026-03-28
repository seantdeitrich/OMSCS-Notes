## Analog to Digital Conversion
- Analog is continuous, while digital is discrete
- Analog becomes digital via sampling, and no information is lost in the process
- **Sampling Theorem**
	- The sampling rate must be fast enough to capture whatever the highest frequency is that we're trying to record
	- This is defined by the **Nyquist** frequency
	- The Sampling frequency must be at least twice as fast as the highest recorded frequency
	- Aliasing and sound distortion can occur if the Nyquist sampling frequency isn't met
- The original analog signal is reconstructed from the digital capture, so that the playback is always analog
	- This is accomplished with Digital to Analog converters (DAC)
- **Quantization** 
	- A sample will represent the amplitude of the signal at a given point in time
	- The analog value of the amplitude is converted to a discrete value
	- In short, Sampling = Time, while Quantization is the value
	- Sampling and Quantization together represent an acoustic event
	- When the quantization doesn't match up with the real value, the result is an audible noise
	- Example of a quantization error (blue dots are recorded value, red curve is actual)
	- ![](../Images/Pasted%20image%2020260322111128.png)

## Psychoacoustics
- The study of human perception through hearing
- Human hearing is designed for self defense, but is quirky and complicated
- The normal human hearing range is 20-20KHz, but diminishes with age
- Perception of intensity (loudness) is measured in decibels, which is a logarithmic scale
- Monaural cues are used for distance perception and crude localization
- Binaural cues are used to detect time differences (echoes) and phase differences
## Audio for Games
- 3D Audio requires a position and orientation for both the listener and sound source
- Software can introduce timing delays to left and right channels
- Volume adjustments can also be applied per channel
- Both help to create a 3D effect
- Usually audio files have the same volume despite representing different things (bee vs airplane)
- **Attenuation** helps resolve this issue by setting an equation that relates distance to volume
- Doppler can be enabled and exaggerated for moving audio sources
- Environmental effects:
	- Reverb, reflections, echo, band pass effects, transitions between areas, etc.
	- Built into unity as presets to help diversify the soundscape
- 

## Quiz Notes:
Question 1:
- [ ] Reducing bit depth per digital audio sample results in reduced quantization noise during reproduction
- [x] Individuals can estimate the position of a familiar and continuous/repeating sound with one ear
- [x] The external part of the ear (pinna) helps front/back disambiguation and vertical localization of a sound source
- [x] Increasing the number of digital audio samples per unit of time increases the maximum frequency of sound that can be captured and reproduced accurately
- [ ] Sound waves of equal amplitude but different frequencies are perceived as all being the same loudness
- [x] Individuals can perceive the time delay between a sound arriving at their ear nearer to the source and their further ear. This binaural cue can be used to aid in localization of the sound source and can be simulated in a video game

Question 2:
- [x] Mix the track down to mono
- [ ] Select the largest possible subset of the source track for the loop
- [x] Always set the starting and ending point at a zero crossing as the loop section is adjusted
- [ ] Always set the starting point at a maximum amplitude sample and ending point at minimum amplitude sample as the loop section is adjusted
- [x] Listen carefully with headphones or studio monitor speakers for anomalies, particularly at the loop over point. Continuously adjust to minimize detectable issues until satisfied.
- [x] Define a loop length that balances repetitiveness with impact on storage requirements of the game