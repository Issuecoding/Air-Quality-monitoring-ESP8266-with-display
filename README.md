# IoT-Air-Quality-monitoring-with-LCD
Using ESP8266, MQ135 and SH1106 

I wanted to create the AQ monitoring system shown at the website below, but I found out after some tinkering around and getting frustrated, that the display I had was in fact not the same as in the website, albeit looks the same.

The original website: https://how2electronics.com/iot-air-quality-index-monitoring-esp8266/

Parts used:
1. Breadboard
2. 7 cables
3. MQ-135 gas sensor
4. ESP8266
5. SH1106 128x64 OLED display, not to be confused with similarly looking SSD1306 by Adafruit!

Because the SH1106 and SSD1306 are not the same, they require different libraries to work. The library to use for the SH1106 is U8g2

![image](https://github.com/Issuecoding/IoT-Air-Quality-monitoring-with-display/assets/148871637/7e709994-95c0-4413-9765-1d019fd8b5b6)
