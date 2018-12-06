---
title: Tricks for Wechat Tiaoyitiao Mini Program
categories: Python
tags: [Python, Arduino]
description: >-
  There are too many tricks for Wechat "Tiaoyitiao" mini program. Here I give a
  method to play the game automatically. Different from the older software
  proposal, I added the Arduino module to imitate the behaviors of the human.
date: 2018-05-11 16:32:36
---
## How does it act?

![](https://media.githubusercontent.com/media/recursively/recursively.github.io/hexo/source/pics/IMG_2723.GIF)

## Distinguish the Screenshot

First of all, we need to work out the program to distinguish the chess piece and board, furthermore, finding the distance between the chess piece and the target position and calculating the pressing time. It's definitely a complicated process. I used the open-source project https://github.com/wangshub/wechat_jump_game. We should connect the mobile phone to the computer with USB and make sure that we have installed android adb debug command so that the mobile phone can take screenshots via the program for the next manipulation.



## Control the Servo With Arduino

I wrote the C program with Arduino IDE, the Arduino serial only accepts string type input, for convenience I use the char_to_int(char i) function to transform the input type. The servo moves to an angle decided by the .write(angle) method, the parameter angle can be different in other situations. The delay time is associated with pressing time based on the distance between the target position and chess piece.

```c
#include <Servo.h>

Servo myservo;

int char_to_int(char i)
{
    switch(i)
    {
        case '0':return 0;
        case '1':return 1;
        case '2':return 2;
        case '3':return 3;
        case '4':return 4;
        case '5':return 5;
        case '6':return 6;
        case '7':return 7;
        case '8':return 8;
        case '9':return 9;
        default:return 0;
    }
}

void setup() {
    Serial.begin(9600);
    pinMode(10, OUTPUT);
    myservo.attach(10);
    myservo.write(0);
    delay(1000);
}

void loop() {
    char a, b, c;
    int i;
    while(!Serial.available());
    if(Serial.available())
    {
        a = Serial.read();
        delay(3);
    }
    if(Serial.available())
    {
        b = Serial.read();
        delay(3);
    }
    if(Serial.available())
    {
        c = Serial.read();
        delay(3);
    }
    if(b == NULL)
    {
        i = char_to_int(a);
        Serial.println(i, DEC);
    }
    else if(c == NULL)
    {
        i = char_to_int(a) * 10 + char_to_int(b);
        Serial.println(i, DEC);
    }
    else
    {
        i = char_to_int(a) * 100 + char_to_int(b) * 10 + char_to_int(c);
        Serial.println(i, DEC);
    }
    i = i*23;

    myservo.write(60);
    delay(i);

    myservo.write(0);
}

```



## Control the Arduino Serial

In order to control the Arduino serial input value, we can use the serial monitor embedded in the Arduino IDE. And we can also use python to Implement the same functionality.

```python
#!/usr/bin/env python  
#coding=utf-8  
import serial 
#from serial import * 
import time  
import threading  
import glob  
  
inhead = 'RECV'      
outhead = 'SEND'     
      
class SerialData(threading.Thread):
    def __init__(self):  
        threading.Thread.__init__(self)     
    def open_com(self, port, baud):         
        self.ser = serial.Serial(port, baud, timeout = 0.5)  
        return self.ser  
    def com_isopen(self):                       
        return self.ser.isOpen()  
    def send_data(self, data, outhead = outhead):   
        self.ser.write(outhead + data)    
    def next(self):                                      
        all_data = ''                             
        #if inhead == self.ser.read(1) :  
        all_data =  self.ser.readline()     
        return  all_data  
    def close_listen_com(self):                 
        return self.ser.close()  
      
if __name__ == '__main__':  
    try:  
        rec_data = SerialData()                 
        allport = glob.glob('/dev/ttyACM*')  
        port = allport[0]                             
        baud = 9600    
        openflag = rec_data.open_com(port, baud) 
        if openflag:  
            print 'i open %s at %s successfully!'%(allport[0], baud)  
        rec_data.send_data('90')
       
        rec_data.close_listen_com()
          
    except KeyboardInterrupt:             
        rec_data.close_listen_com()
        if not rec_data.com_isopen():
            print('serial closed')
```



## Combine the Codes

The last step is combining the picture distinguishing code and the serial controlling code. I've run the code with python3.6 and the whole process performed well.
