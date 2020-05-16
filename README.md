# m5stack-sigfox
Sigfox RF monitoring for M5Stack Basic

## Components and Supplies

- [M5Stack Basic](https://m5stack.com/collections/m5-core/products/basic-core-iot-development-kit)
- [M5Stack Proto Module](https://m5stack.com/collections/m5-module/products/proto-module)
- [Sigfox Breakout Board (BRKWS01)](https://yadom.eu/carte-breakout-sfm10r1.html)

![20200516a](https://user-images.githubusercontent.com/9893802/82108211-e7868580-9767-11ea-975b-3483554b268a.png)

## Connect Sigfox Breakout board (BRKWS01)

Connect Pin 16/17 on M5Stack to Tx/Rx on BRKWS01, then provide 3.3V power. 
![20200516b](https://user-images.githubusercontent.com/9893802/82108791-2c142000-976c-11ea-847e-b50579fe7ac7.png)

## Configure Downlink data

To get a downlink data, you configure downlink setting on your device type in [Sigfox backend cloud](https://backend.sigfox.com/).
![20200516c](https://user-images.githubusercontent.com/9893802/82108855-ae044900-976c-11ea-937a-ab1746944eb3.png)
In this case, ID and RSSI on the base station that received device message are needed, so you select **DIRECT** as Downlink mode. And {tapid} and {rssi} variables must be included in a downlink data definition.

## Sample Code
```
#include <M5Stack.h>
void setup() {
  M5.begin(true, false, true);

  M5.Power.begin();

  Serial.begin(9600);
  Serial2.begin(9600, SERIAL_8N1, 16, 17);

  M5.Lcd.clear(BLACK);
  M5.Lcd.setTextColor(YELLOW);
  M5.Lcd.setTextSize(2);
  M5.Lcd.setCursor(65, 10);
  M5.Lcd.println("Sigfox RF monitor");
  M5.Lcd.setCursor(0, 35);
  M5.Lcd.println("A: Send Message");
  M5.Lcd.println("B: Send Message with DL");
  M5.Lcd.println("C: Check Device ID");
  M5.Lcd.setTextColor(RED);
}

void loop() {
  if (Serial2.available()) {
    displayResults(Serial2.readString());
  }
  M5.update();

  if (M5.BtnA.wasReleased()) {
    M5.Lcd.println("Send Message.");
    Serial2.println("AT$SF=1234");
  } else if (M5.BtnB.wasReleased()) {
    M5.Lcd.println("Send Message with Ack.");
    Serial2.println("AT$SF=5678,1");    
  } else if (M5.BtnC.wasReleased()) {
    M5.Lcd.print("Device ID: ");
    Serial2.println("AT$I=10");
  }
}

void displayResults(String ack)
{
  M5.Lcd.println(ack);
  int i = ack.indexOf("RX=");
  if (i >= 0) {
    ack.replace(" ", "");
    String bs = ack.substring(i + 6, i + 11);
    String rs = ack.substring(i + 15);
    signed int rssi = (int16_t)(strtol(rs.c_str(), NULL, 16));
    M5.Lcd.print("BSID: ");
    M5.Lcd.println(bs);
    M5.Lcd.print("RSSI: ");
    M5.Lcd.println(rssi);
  }
}
```
When you push a button 'B', Send frame command with ack request (AT$SF=[payload],1) is sent to Sigfox module via Serial2. 
After approximately 30 mins, Sigfox module will receive a downlink message including acutual values of {tapid} and {rssi}. 

![20200516d](https://user-images.githubusercontent.com/9893802/82109210-3257cb80-976f-11ea-811a-f9a4ad515144.jpeg)

## With M5Stack Proto module
If you have M5Stack Proto module, you can build a Sigfox breackout board on the proto module.
![20200516e](https://user-images.githubusercontent.com/9893802/82109263-89f63700-976f-11ea-9102-36dbc97183a6.png)

Twitter: [@ghibi](https://twitter.com/ghibi)
