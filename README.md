# Lightning-Detector
AS3935 based lightning detector with TFT display and Teensy 4.0 microcontroller
AS 3935 Franklin Lightning Detector Application that works
This Teensy controlled lightning detector application actually works! It took quite a while to figure out all the gotchas that were causing failures with my previous implementations. Also using the 320x240 color LCD display  wasn’t so easy because addressing it kept bleeding electrical noise into the lightning detector board via the SPI bus. I haven’t totally finished the final version yet but this worked reliably during multiple storms this last summer. It could sense storms approximately 20 miles away even when using  indoor mode (more on that later).  

I’m an amateur radio operator and electrical engineer and still found using the AS 3935 difficult to implement. It took a year of perseverance to learn all the lessons needed to make this work. 

Several key factors needed to be learned before success. One factor was to use 3.3V parts. 5V controllers such as the Arduino nano and Mega caused detector interference. I ultimately chose the Teensy 4.0 as it is 3.3V and operates at such a high clock frequency (600 Mhz) that it didn’t really interfere with the AS3935’s 500kHz tank circuit band width. I also found that trying to use I2C for the AS-3935 was not reliable. Sometimes it worked some times not.  This caused a lot of wasted time (testing) so I finally ditched I2C and implemented the SPI interface.  I also wanted a fancy TFT color display for lightning  detection , range display , and storm strength indications. I also added a disturber detection function as I found some real lightning strikes get classified as a disturber probably because the strike waveform is outside of the AMS secret detection algorithm requirements. Initially adding the TFT display caused 500kHz interference to the detector when I was writing to the display. I also added a red LED on the detectors interrupt line so I could easily see when the interrupt line was indicating an event when none should be present. The SPI communication trick here was to isolate the Sparkfun AS 3935 SPI pins from the TFT SPI pins. This is accomplished with the 74HC244 buffer IC. The HC version is 3.3V compatible, so don’t use a 74HCT 244 or 74244 as it won’t work. I also found I had to move the Spark fun AS 3935 PCB away from the other components. Six  inches minimum separation seemed to be OK.   In the AMS data sheet they recommend keeping the AS-3935 away from switching power supplies and other electrical gear. This little AMS chip has a 500kHz tank circuit that is surprisingly sensitive to ANY local 500kHz noise. While doing my real lightning testing I was turning on my short wave receiver to also watch and hear 500kHz lightning signals (for strike verification). { A short wave receiver tuned to 500kHz will indicate a static crash sound during lightning strikes. ) Every time I turned on my SW receiver the lighting detector stopped working . I finally tracked it down to the switching power supply that runs the SW receiver. Even when placed 100 ft away from the detector it still blocked the lightning signals. However, the lightning detector would work fine up to about 2 ft away from my Dell desktop computer. 
I also found a coding error in the Sparkfun AS3935 library. That error will not let you ever change the AS3935 sensitivity to outdoors. So if you want outdoors mode to work you will need to fix SparkFun’s  AS3935.CPP code 
To fix the SparkFun_AS3935_Lightning_Detector_Arduino_Library look in the sparkfun library src folder and open the Sparkfun_AS3935.cpp file with MS notepad. Find the section :
// REG0x00, bits [5:1], manufacturer default: 10010 (INDOOR).
// This function changes toggles the chip's settings for Indoors and Outdoors.
void SparkFun_AS3935::setIndoorOutdoor(uint8_t _setting)
{
    if (((_setting != INDOOR) && (_setting != OUTDOOR)))
       return
--------------------------------------
Change the if statement line  to : if (((_setting != INDOOR) || (_setting != OUTDOOR)))
Save the file back to the library. 


Next I plan on putting my breadboard into a metal box with the Sparkfun AS3935 outside the metal chassis.  I also plan on fixing the problems I had with range to storm and storm intensity. The numbers are often  incorrect and I suspect I have a read conflict going on. Occasionally they are correct. It may also be a register read timing issue. As we slide into autumn I’m not as ambitious to fix this problem.  Another minor issue I see is when there are multiple lightning strikes. The AS 3935 can’t keep up with numerous rapid strikes, so it misses some strikes ( I guess that’s ok as long as it see most of them!) 
A few more notes:
•	You should be able to use other 320x240 3.3V TFT displays without a problem. I think the Adafruit display is now obsolete. 
•	While doing testing I found that The piezo electric candle lighters don’t work well. You’ll get disturbers and very rarely a simulated strike. 
•	If you get the lighter too close to the AS 3935 IC you can destroy it. I had this happen and the failure mode is insidious. The IC still appears to work as you can read and write registers, but it  won’t catch real lightning as the receiver circuitry was what got destroyed. After a few real thunder storms passed I realized what happened as nothing was detected.  
•	I bought and used a SEN-39002 lightning simulator. It doesn’t work well at all. I think it’s because the SEN-39002 is a shield that sits on top of an Arduino UNO and the UNO emits RFI (radio frequency interference) along with the simulated lightning signals so the UNO RFI masks the simulated signals. I tried various fixes but nothing worked well and “Playing with fusion”  didn’t answer my emails. It does occasionally trigger the Sparkfun AS3935 so I haven’t tossed it yet. 
•	A lot of hobbyists have complained the AS 3935 doesn’t work. It does work fine but you really need to work around the RFI issues which I have done. It was really exciting watching the multiple lightning strikes get detected from several big storms that I monitored this last summer. I’ll post a video of the TFT on Youtube along with this write up. 

The Youtube Video showing visual details and the lightning detector operating is located at  https://youtu.be/ibwPA6T7BcI 


