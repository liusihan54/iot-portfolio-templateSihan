At first, no waveform appeared in the web interface. One possible cause was that the signal wire was connected to a different GPIO pin from the input pin configured in the firmware. After checking the pin definitions and adjusting the wiring to match them, we started another capture and successfully displayed the waveform.

I do not remember the exact adjustment, so this is a possible explanation rather than a confirmed diagnosis. The experience taught me to check the physical GPIO connections against the firmware settings before assuming that the software is faulty.
