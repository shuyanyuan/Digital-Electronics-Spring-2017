#Processing code:

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


#Arduino code:
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

