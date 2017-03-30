### examples:

Musical Instrument: A COLLECTIVE INSTRUMENT CAPTURING BREATHE WITH PAPER WINDMILLS

Cata Sopros is interactive sound installation running on Arduino Uno and created by Elas Duas, the project is a collective musical instrument made with paper windmills transforming the users’ breathe into sounds:

The windmills have inbuilt electret microphones that were connected to an Arduino Uno. The sensor data was then sent to MaxMSP and the sounds were played with Ableton Live. The video was shot at the cloister of the beautiful Alberto Sampaio museum in Guimarães, Portugal.

Link:https://vimeo.com/108800391

### Sensors Research:

Distance Sensor

The distance sensor measures distance by shining a beam of infrared light and uses a phototransitor to measure the intensity of the light that bounces back. The effective distance measuring range for this proximity sensor is 10-80cm. If an object is closer than the shortest distance, it reports a significantly higher analog output and is inconsistent with expected in-range results.

Tilt Sensor

A tilt sensor can measure the tilting in often two axes of a reference plane in two axes. In contrast, a full motion would use at least three axes and often additional sensors. One way to measure tilt angle with reference to the earths ground plane, is to use an accelerometer. Typical applications can be found in the industry and in game controllers.

PIR Sensor

A passive infrared sensor (PIR sensor) is an electronic sensor that measures infrared (IR) light radiating from objects in its field of view. They are most often used in PIR-based motion detectors.

#Concepts:

1.Music egg carton

Users can use hand movements to feel the sound of eggs, each egg has an especial tone of sound, so that users can notice the different when interacting the egg carton. By using PIR sensor and put a LED inside the egg, when a hand pass through it, the egg will be brighter and the produce a sound.

.2. Music balloon

By using the RGB color sensor, different color of balloon has an especial sound, users can by touching different color of balloon to create a song.



# Final music instrument
This is a music instrument that allows users play the music backward or forward by putting the hand to different position of the music cover. Using distance sensor to detect the distance of the hand, if user's hand moves far from the sensor, the music will play forward, otherwise, the music will play backward if user's hand moves closer to the sensor.

![alt text](https://raw.githubusercontent.com/shuyanyuan/Digital-Electronics-Spring-2017/master/MusicInstrument/Snip20170330_3.png)
# Processing code:

import processing.sound.*;
import processing.serial.*; 
import ddf.minim.*;
import ddf.minim.signals.*;

Serial myPort;        // The serial port

SoundFile file;

Minim minim;
AudioOutput out;
AudioPlayer player;

float freq ;
float amp ;
float distance ;

void setup(){
  size(900,600);
   background(255);
  
  myPort = new Serial(this, Serial.list()[1], 9600);
  myPort.bufferUntil('\n');
  
  minim = new Minim(this);
  out = minim.getLineOut(Minim.STEREO);
  
 
  file =new SoundFile(this, "IFollowRivers.mp3");
  //player = minim.loadFile("BlackBeatles.mp3");
  
  file.loop();  

}

void draw(){
  
  
  //file.rate(map(mouseX, 0, width, 0.25, 4.0));
  //file.amp(map(mouseX, 0, width, 0.25, 4.0));
  
  file.rate(freq);
  //file.amp(freq);
  
 
  //file.amp(distance);
  

  
} 

void serialEvent (Serial myPort) {
  String inString = myPort.readStringUntil('\n');
  if (inString != null) {
    println(inString);
    inString = trim(inString);
    int sensors[] = int(split(inString, ','));


    if (sensors.length == 1) {

      //freq = map(sensors[0], 0, 1023, 110, 880);
      
      //file.rate(map(mouseX, 0, width, 0.25, 4.0));
      freq =  map(sensors[0], 0, 30, 1, 8);
      //distance = map(sensors[0], 0, 25, 1, 4);
      //amp = map(sensors[0], 0, 50, 0, 4);
  
      
      //print(rate);
      //println(distance);
      println(freq);
     
    }
  }
}

void stop()
{
  out.close();
  minim.stop();
  super.stop();
}


# Arduino code:
#define trigPin 7
#define echoPin 6
#define led 13
#define led2 12
#define led3 11
//#define led4 10
//#define led5 9
//#define led6 8
//#define buzzer 3

//int sound = 250;


void setup() {
  // initialize the serial communication:
  Serial.begin (9600);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(led, OUTPUT);
  pinMode(led2, OUTPUT);
  pinMode(led3, OUTPUT);
//  pinMode(buzzer, OUTPUT);

}

void loop() {
  // send the value of analog input 0:
  Serial.println(analogRead(A0));
  
  long duration, distance;
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  duration = pulseIn(echoPin, HIGH);
  distance = (duration / 2) / 29.1;

  if (distance <= 20) {
    digitalWrite(led, HIGH);
//    sound = 250;
  }
  else {
    digitalWrite(led, LOW);
    }
  if (distance < 15) {
    digitalWrite(led2, HIGH);
//    sound = 260;
  }
  else {
    digitalWrite(led2, LOW);
 }
 if (distance < 10) {
   digitalWrite(led3, HIGH);
//    sound = 270;
  }
  else {
   digitalWrite(led3, LOW);
 }


  if (distance > 20 || distance <= 0) {
    Serial.println("Out of range");
//    noTone(buzzer);
  }
  else {
    Serial.print(distance);
     Serial.print(',');
     Serial.println(analogRead(A1));
//    tone(buzzer, sound);

  }
  delay(500);
}

