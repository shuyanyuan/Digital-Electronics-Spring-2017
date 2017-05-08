# Final project

## Proposal
For the final project, I wanted to create an installation that is used physical input and output instead of screen based visual output.
After the idea brainstorming, I came up an idea to create a lit bomb object, the flames will be keep sparkling and flashing, in order to extinguish the flames, people can use the real action in reality to blow out the flame. The bomb and the sparkle is a metaphor and signifier to indicate people use the blow a flame action to interact with the installation.

## Materials Used
FLORA - for minimaling the components and eventually get rid of a computer and breadbords
Neopixel strip - to represent the flame, program and connect to the sensor
Electret microphone (100Hz–10kHz) Amplifier - detect the analog sound input to control the neopixel
El wire- flexible to make a bomb sharpe
Laser cut- use AI to draw the outline and frame and laser cut it
Soldering- solder the piecemeal neopixels together

## Reaseach
Guide for Microphone Sound Sensor with Arduino
https://randomnerdtutorials.com/guide-for-microphone-sound-sensor-with-arduino/

Sound reactive room lighting using Arduino and LED strips
https://votecharlie.com/blog/2016/02/sound-reactive-room-lighting-using-arduino-and-led-strips.html

Sound sensitive lights w/ sound sensor & Arduino
http://www.princetronics.com/sound-sensitive-lights-w-sound-sensor-arduino/

Sound sensitive NeoPixel cap
https://learn.adafruit.com/vu-meter-baseball-hat/add-leds-to-hat

## Circuit Diagram
![alt tag](https://github.com/shuyanyuan/Digital-Electronics-Spring-2017/blob/master/finalProject/led_strips_flora-diagram.jpg
)
## Schematic
![alt tag](https://github.com/shuyanyuan/Digital-Electronics-Spring-2017/blob/master/finalProject/schematic.jpg)

## Laser Cut
![alt tag](https://github.com/shuyanyuan/Digital-Electronics-Spring-2017/blob/master/finalProject/laser_cut.jpeg)

## Prototype
![alt tag](https://github.com/shuyanyuan/Digital-Electronics-Spring-2017/blob/master/finalProject/prototype.jpeg)
![alt tag](https://github.com/shuyanyuan/Digital-Electronics-Spring-2017/blob/master/finalProject/soldering.jpeg)
![alt tag](https://github.com/shuyanyuan/Digital-Electronics-Spring-2017/blob/master/finalProject/construsting.jpeg)

##Final 
![alt tag](https://github.com/shuyanyuan/Digital-Electronics-Spring-2017/blob/master/finalProject/final.jpeg)

## Code on Arduino 1.0.5
```
/*
Hardware requirements:
 - Most Arduino or Arduino-compatible boards (ATmega 328P or better).
 - Adafruit Electret Microphone Amplifier (ID: 1063)
 - Adafruit NeoPixel Digital LED strip (ID: 1138)
 - External battery for portable use (else power through USB or adapter)
Software requirements:
 - Adafruit NeoPixel library

Connections:
 - 3.3V to mic amp +
 - GND to mic amp -
 - Analog pin to microphone output (configurable below)
 - Digital pin to LED data input (configurable below)

  Written by Shuyan Iris Yuan.  
*/


//Import Adafruit NeoPixel library
#include <Adafruit_NeoPixel.h>

#define N_PIXELS  20  // Number of pixels in strand
#define MIC_PIN   A9  // Microphone is attached to this analog pin
#define LED_PIN    6  // NeoPixel LED strand is connected to this pin
#define DC_OFFSET  0  // DC offset in mic signal - if unusure, leave 0
#define NOISE     10  // Noise/hum/interference in mic signal
#define SAMPLES   60  // Length of buffer for dynamic level adjustment
#define TOP       (N_PIXELS + 2) // Allow dot to go slightly off scale
#define PEAK_FALL 4  // Rate of peak falling dot
//int val =0;      //Mic value

byte
  peak      = 0,      // Used for falling dot
  dotCount  = 0,      // Frame counter for delaying dot-falling speed
  volCount  = 0;      // Frame counter for storing past volume data
int
  vol[SAMPLES],       // Collection of prior volume samples
  lvl       = 10,      // Current "dampened" audio level
  minLvlAvg = 0,      // For dynamic adjustment of graph low & high
  maxLvlAvg = 512;
  val =0;             //Mic value
  
Adafruit_NeoPixel
  strip = Adafruit_NeoPixel(N_PIXELS, LED_PIN, NEO_GRB + NEO_KHZ800);


//setup function: initializes and sets the initial values
void setup() {


  memset(vol, 0, sizeof(vol));
  //Command NeoPixel to start 
  strip.begin();
}

//the loop() function does precisely what its name suggests, and loops consecutively
void loop() {
  uint8_t  i;
  uint16_t minLvl, maxLvl;
  int      n, height;

//Read the microphone data from serial monitor


val =analogRead(MIC_PIN);     //Read the analog value of microphone
  Serial.print("mic: ");     
  Serial.println(val);       //Print the analog value
  
 
 

  n   = analogRead(MIC_PIN);                        // Raw reading from mic 
  n   = abs(n - 512 - DC_OFFSET);                   // Center on zero
  n   = (n <= NOISE) ? 0 : (n - NOISE);             // Remove noise/hum
  lvl = ((lvl * 7) + n) >> 3;    // "Dampened" reading (else looks twitchy)

  // Calculate bar height based on dynamic min/max levels (fixed point):
  height = TOP * (lvl - minLvlAvg) / (long)(maxLvlAvg - minLvlAvg);

  if(height < 0L)       height = 0;      // Clip output
  else if(height > TOP) height = TOP;
  if(height > peak)     peak   = height; // Keep 'peak' dot at top

  // white color pixels based on gradient
  for(i=0; i<N_PIXELS; i++) {
    if(i >= height)               strip.setPixelColor(i,255,255,255) ;
    
    else strip.setPixelColor(i,   0,   0, 0);
    
  }



  // Draw peak dot  
  if(peak > 0 && peak <= N_PIXELS-1) strip.setPixelColor(i,255,255,255) ;
  strip.show(); // Update strip

// Every few frames, make the peak pixel drop by 1:

    if(++dotCount >= PEAK_FALL) { //fall rate 
      
      if(peak > 0) peak--;
      dotCount = 0;
    }



  vol[volCount] = n;                      // Save sample for dynamic leveling
  if(++volCount >= SAMPLES) volCount = 0; // Advance/rollover sample counter

  // Get volume range of prior frames
  minLvl = maxLvl = vol[0];
  for(i=1; i<SAMPLES; i++) {
    if(vol[i] < minLvl)      minLvl = vol[i];
    else if(vol[i] > maxLvl) maxLvl = vol[i];
  }
  // minLvl and maxLvl indicate the volume range over prior frames, used
  // for vertically scaling the output graph (so it looks interesting
  // regardless of volume level).  If they're too close together though
  // (e.g. at very low volume levels) the graph becomes super coarse
  // and 'jumpy'...so keep some minimum distance between them (this
  // also lets the graph go to zero when no sound is playing):
  if((maxLvl - minLvl) < TOP) maxLvl = minLvl + TOP;
  minLvlAvg = (minLvlAvg * 63 + minLvl) >> 6; // Dampen min/max levels
  maxLvlAvg = (maxLvlAvg * 63 + maxLvl) >> 6; // (fake rolling average)

}

// Input a value 0 to 255 to get a color value.
// The colors are a transition r - g - b - back to r.
uint32_t Wheel(byte WheelPos) {
  if(WheelPos < 85) {
   return strip.Color(WheelPos * 3, 255 - WheelPos * 3, 0);
  } else if(WheelPos < 170) {
   WheelPos -= 85;
   return strip.Color(255 - WheelPos * 3, 0, WheelPos * 3);
  } else {
   WheelPos -= 170;
   return strip.Color(0, WheelPos * 3, 255 - WheelPos * 3);
  }
}
```
