# IoT-Air-Quality-monitoring-with-LCD
Using ESP8266, MQ135 and SH1106 

I wanted to create the AQ monitoring system shown at the website below, but I found out after some tinkering around and getting frustrated, that the display I had was in fact not the same as in the website, albeit looks the same.

The original website: https://how2electronics.com/iot-air-quality-index-monitoring-esp8266/

Parts used:
1. Breadboard
2. 7 connecting wires
3. MQ-135 gas sensor
4. ESP8266
5. SH1106 128x64 OLED display, not to be confused with similarly looking SSD1306 by Adafruit!

Because the SH1106 and SSD1306 are not the same, they require different libraries to work. The library to use for the SH1106 is U8g2

![image](https://github.com/Issuecoding/IoT-Air-Quality-monitoring-with-display/assets/148871637/7e709994-95c0-4413-9765-1d019fd8b5b6)

How did I figure out I was not using the correct library? Thanks to [this comment here](https://www.reddit.com/r/arduino/comments/u9owsr/i_set_up_my_128x64_oled_to_draw_a_rectangle_and/i5t95m0/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) on Reddit.


## Additional things to know
This code allows you to upload everything to Thingspeak so you can monitor it remotely too!

## The code

```
#include <ESP8266WiFi.h>
#include <Arduino.h>
#include <U8g2lib.h>
#include "MQ135.h"

#ifdef U8X8_HAVE_HW_SPI
#include <SPI.h>
#endif
#ifdef U8X8_HAVE_HW_I2C
#include <Wire.h>
#endif


U8G2_SH1106_128X64_NONAME_F_HW_I2C u8g2(U8G2_R0, /* reset=*/ U8X8_PIN_NONE);

 
String apiKey = "14K8UL2QEK8BTHN6"; // Enter your Write API key from ThingSpeak
const char *ssid = "Potato";     // replace with your wifi ssid and wpa2 key
const char *pass = "123456789";  // replace with your wifi password
const char* server = "api.thingspeak.com";
 
WiFiClient client;
 

/* setup(void) {
  u8g2.begin();
}*/

void setup()
{
  u8g2.begin();
  Serial.begin(115200);
  u8g2.clearBuffer();          // clear the internal memory
  delay(10);
 
  u8g2.setFont(u8g2_font_ncenB08_tr);  // choose a suitable font
  u8g2.setCursor(1,10);
  u8g2.print("Connecting to ");
  u8g2.setCursor(1,22);
  u8g2.print(ssid);
  u8g2.sendBuffer();          // transfer internal memory to the display
  delay(1000); 
  
  WiFi.begin(ssid, pass);
 
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }
    Serial.println("");
    Serial.println("WiFi connected");

    u8g2.clearBuffer();          // clear the internal memory
    delay(10);
    u8g2.drawStr(1,10,"WiFi connected");
    u8g2.sendBuffer();          // transfer internal memory to the display
    delay(4000); 

}
 
 
  void loop()
  {
    MQ135 gasSensor = MQ135(A0);
    float air_quality = gasSensor.getPPM();
    Serial.print("Air Quality: ");  
    Serial.print(air_quality);
    Serial.println("  PPM");   
    Serial.println();

    u8g2.clearBuffer();          // clear the internal memory
    delay(10);
    u8g2.setFont(u8g2_font_ncenB08_tr);  // choose a suitable font

    u8g2.setCursor(1,10);
    u8g2.print("Air Quality Index");
    u8g2.sendBuffer();          // transfer internal memory to the display
    delay(1000);

    /* Air quality number */
    u8g2.setFont(u8g2_font_ncenB08_tr);  // choose a suitable font
    u8g2.setCursor(1,20);
    u8g2.print(air_quality);
    u8g2.setCursor(40,20);
    u8g2.print("PPM");          // PPM label, need to add on same line as number though
    u8g2.sendBuffer();          // transfer internal memory to the display
    delay(1000);

 
 
    if (client.connect(server, 80)) // "184.106.153.149" or api.thingspeak.com
  {
    String postStr = apiKey;
    postStr += "&field1=";
    postStr += String(air_quality);
    postStr += "r\n";
    
    client.print("POST /update HTTP/1.1\n");
    client.print("Host: api.thingspeak.com\n");
    client.print("Connection: close\n");
    client.print("X-THINGSPEAKAPIKEY: " + apiKey + "\n");
    client.print("Content-Type: application/x-www-form-urlencoded\n");
    client.print("Content-Length: ");
    client.print(postStr.length());
    client.print("\n\n");
    client.print(postStr);
    
    Serial.println("Data Send to Thingspeak");
  }
    client.stop();
    Serial.println("Waiting...");
 
    delay(2000);      // thingspeak needs minimum 15 sec delay between updates.
}

```
