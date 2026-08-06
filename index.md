Hello! For my project I chose the Gesture Controlled Robot! It is a robot car that is controlled by gestures created by a hand module. The robot and hand are connected with bluetooth, and programmed on Arduino IDE with C++. An accelerometer on the hand module measures tilts in the x and y direction, prompting the robot car to move it's motors in certain directions.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Mari I | Forest Hills High School | Electrical Engineering | Incoming Senior

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# Final Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/VcYvCaiRvsA?si=HAfj8h7XWsztrqDr" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Since my last milestone I have fully completed the base project! The robot and hand module operate wirelessly with no issues. For modifications I created a glove to place the hand module on. This is so that I can wear it, and it looked cooler! Another modification I added was a camera and screen. I planned to have a ESP-32 Camera mounted on the robot to stream where the robot was moving, and a ILI9341 TFT screen to display it from a distance. 

The most challenging aspect of this milestone was debugging the screen and camera. I had to create its own wifi network with 2.4Ghz, and configuring between the camera and display was excruciatingly hard. Whether it was finding the right resolution, quality or frame rate I went through hundreds of lines of code trying to find that one error that was distorting my screen. In the end, I was able to find the bugs and have my screen display the camera's live footage. This took a toll on my mental state was well as my hope but It was able to teach a great lesson of perseverance. The struggles I faced throughout this project made me realize how much I love the satisfaction of overcoming it.


# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/_Y3Ua7yF0xw?si=w-l5seFiVvi7H6WA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Since my last milestone, I successfully connected the Bluetooth modules so the hand controller can communicate wirelessly with the robot. I also added an accelerometer to the controller, allowing it to detect my hand movements. After sending the X and Y coordinates to the robot, I programmed the Arduino Uno to respond to those values and move accordingly. To make the controller completely portable, I powered it with a 9-volt battery.

The biggest challenges were wiring the electronics, getting the Bluetooth modules to communicate, and debugging the accelerometer. For my next milestone, I plan to turn the controller into a wearable glove and add a speed boost feature to the robot to improve its performance. Thank you for watching!

# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/l__13xIZgQo?si=VUcc7zlaqovNP_uv" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

  For my first milestone, I focused mainly on the hardware. I followed the tutorial to build the robot chassis by attaching the wheels and motors. I also wired the motors, motor driver, and Arduino UNO together.

I soon realized that the battery clips had the wrong connector to attach to my motor driver. While I was waiting for the correct parts to arrive, I was able to test each motor individually. Then, to get the Arduino UNO to control the motor driver, I wrote a sketch that allowed the motors to move forward. To power the UNO, I used the battery pack with the barrel jack attachment.

To become more familiar with C++, I programmed my robot to perform a sequence of movements: repeat four times by moving forward and then turning right, perform a spin, move backward, and then move forward again. Once my new battery clip arrived, I completed many trial runs to perfect the sequence.

That is my progress so far! Next, I would like to work on connecting the Bluetooth modules and perfecting the handheld controller module.


# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

# Hand Module Code

```c++
#include <Wire.h>
#define BT_Serial Serial1

const int MPU_ADDR = 0x68;

int16_t AcX, AcY, AcZ;
int X_value;
int Y_value;
int flag = 0;

void setup() {

  Serial.begin(38400);      // Serial Monitor
  BT_Serial.begin(38400);    // HC-05

  Wire.begin();

  // Wake up MPU6050
  Wire.beginTransmission(MPU_ADDR);
  Wire.write(0x6B);
  Wire.write(0);
  Wire.endTransmission(true);

  delay(500);

  Serial.println("MPU6050 Ready");
}

void loop() {

  readMPU6050();

  Serial.print("X: ");
  Serial.print(X_value);

  Serial.print("  Y: ");
  Serial.println(Y_value);

  // Forward (tilt forward farther)
  if (X_value < 45 && flag == 0) {
    flag = 1;
    BT_Serial.write('f');
    Serial.println("Forward");
  }

  // Backward (tilt backward farther)
  else if (X_value > 135 && flag == 0) {
    flag = 1;
    BT_Serial.write('b');
    Serial.println("Backward");
  }

  // Left
  else if (Y_value < 45 && flag == 0) {
    flag = 1;
    BT_Serial.write('l');
    Serial.println("Left");
  }

  // Right
  else if (Y_value > 135 && flag == 0) {
    flag = 1;
    BT_Serial.write('r');
    Serial.println("Right");
  }

  // Larger neutral zone = stop
  else if (X_value > 60 && X_value < 120 &&
           Y_value > 60 && Y_value < 120 &&
           flag == 1) {
    flag = 0;
    BT_Serial.write('s');
    Serial.println("Stop");
  }
  delay(100);
}

void readMPU6050() {
  Wire.beginTransmission(MPU_ADDR);
  Wire.write(0x3B);
  Wire.endTransmission(false);

  Wire.requestFrom(MPU_ADDR, 6, true);

  AcX = Wire.read() << 8 | Wire.read();
  AcY = Wire.read() << 8 | Wire.read();
  AcZ = Wire.read() << 8 | Wire.read();

  // Convert raw accelerometer values
  X_value = map(AcX, -17000, 17000, 0, 180);
  Y_value = map(AcY, -17000, 17000, 0, 180);

  X_value = constrain(X_value, 0, 180);
  Y_value = constrain(Y_value, 0, 180);

}
```

#Robot Car Code

```c++
#include <SoftwareSerial.h>
SoftwareSerial BT_Serial(2, 3);   // RX, TX

// Motor Driver Pins
#define ENA 5
#define IN1 6
#define IN2 7
#define ENB 10
#define IN3 8
#define IN4 9

char bt_data = 's';

int Speed = 180;

void setup()
{
    Serial.begin(9600);
    BT_Serial.begin(38400);

    pinMode(ENA, OUTPUT);
    pinMode(IN1, OUTPUT);
    pinMode(IN2, OUTPUT);
    pinMode(ENB, OUTPUT);
    pinMode(IN3, OUTPUT);
    pinMode(IN4, OUTPUT);

    Stop();

    delay(500);

    Serial.println("Robot Ready");
}

void loop()
{
    if (BT_Serial.available() > 0)
    {
        bt_data = BT_Serial.read();

        Serial.print("Received: ");
        Serial.println(bt_data);
    }

    if (bt_data == 'f')
    {
        forward();
        Speed = 180;
    }

    else if (bt_data == 'b')
    {
        backward();
        Speed = 180;
    }

    else if (bt_data == 'l')
    {
        turnLeft();
        Speed = 120;
    }

    else if (bt_data == 'r')
    {
        turnRight();
        Speed = 120;
    }

    else if (bt_data == 's')
    {
        Stop();
    }

    analogWrite(ENA, Speed);
    analogWrite(ENB, Speed);

    delay(50);
}

// Forward
void forward()
{
    digitalWrite(IN1, HIGH);
    digitalWrite(IN2, LOW);

    digitalWrite(IN3, LOW);
    digitalWrite(IN4, HIGH);
}

// Backward
void backward()
{
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, HIGH);

    digitalWrite(IN3, HIGH);
    digitalWrite(IN4, LOW);
}

// Turn Right (switched)
void turnRight()
{
    digitalWrite(IN1, HIGH);
    digitalWrite(IN2, LOW);

    digitalWrite(IN3, HIGH);
    digitalWrite(IN4, LOW);
}

// Turn Left (switched)
void turnLeft()
{
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, HIGH);

    digitalWrite(IN3, LOW);
    digitalWrite(IN4, HIGH);
}

//Stop
void Stop()
{
    digitalWrite(IN1, LOW);
    digitalWrite(IN2, LOW);

    digitalWrite(IN3, LOW);
    digitalWrite(IN4, LOW);
}
```

#Camera Code
```c++
#include "esp_camera.h"
#include <WiFi.h>
#include <ArduinoWebsockets.h>

#define CAMERA_MODEL_AI_THINKER

#include "camera_pins.h"

const char* ssid = "esp32net";
const char* password = "123456789";

const char* websockets_server_host = "192.168.4.1"; 
const uint16_t websockets_server_port = 80;

using namespace websockets;
WebsocketsClient client;

void setup() {
  Serial.begin(115200);
  Serial.setDebugOutput(true);
  Serial.println();

  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;
  //init with high specs to pre-allocate larger buffers
  if(psramFound()){
    config.frame_size = FRAMESIZE_QVGA; // 320x240
    config.jpeg_quality = 20;
    config.fb_count = 2;
  } else {
    config.frame_size = FRAMESIZE_QVGA;
    config.jpeg_quality = 20;
    config.fb_count = 1;
  }

  // camera init
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }
  sensor_t *s = esp_camera_sensor_get();
  s->set_vflip(s, 1);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");

  Serial.print("Camera Ready! Use 'http://");
  Serial.print(WiFi.localIP());
  Serial.println("' to connect");

  while(!client.connect("ws://192.168.4.1:80")){
    delay(500);
    Serial.print(".");
  }
  Serial.println("Socket Connected!");  
}

void loop() {
  camera_fb_t *fb = NULL;
  esp_err_t res = ESP_OK;
  fb = esp_camera_fb_get();
  if(!fb){
    Serial.println("Camera capture failed");
    esp_camera_fb_return(fb);
    return;
  }

  size_t fb_len = 0;
  if(fb->format != PIXFORMAT_JPEG){
    Serial.println("Non-JPEG data not implemented");
    return;
  }
Serial.printf("JPEG = %d bytes\n", fb->len);
client.sendBinary((const char*)fb->buf, fb->len);
esp_camera_fb_return(fb);
delay(50);
}
```

#Display Screen Code
```c++
#include <SPI.h>
#include <ArduinoWebsockets.h>
#include <WiFi.h>

#include <TJpg_Decoder.h>
#include <TFT_eSPI.h>

const char* ssid = "esp32net";
const char* password = "123456789";

using namespace websockets;
WebsocketsServer server;
WebsocketsClient client;

TFT_eSPI tft = TFT_eSPI();         // Invoke custom library

bool tft_output(int16_t x, int16_t y, uint16_t w, uint16_t h, uint16_t* bitmap)
{
  // Stop further decoding as image is running off bottom of screen
 if ( y >= tft.height() ) return 0;

 // This function will clip the image block rendering automatically at the TFT boundaries
 tft.pushImage(x, y, w, h, bitmap);

 // This might work instead if you adapt the sketch to use the Adafruit_GFX library
 // tft.drawRGBBitmap(x, y, bitmap, w, h);

 // Return 1 to decode next block
 return 1;
}

void setup() {
 // put your setup code here, to run once:
 Serial.begin(115200);
 delay(1000);
 tft.begin();
 Serial.printf("Display = %d x %d\n", tft.width(), tft.height());

 tft.fillScreen(TFT_RED);
 delay(1000);
 tft.fillScreen(TFT_GREEN);
 delay(1000);
 tft.fillScreen(TFT_BLUE);
 delay(1000);
 tft.setRotation(3);
 tft.setTextColor(0xFFFF, 0x0000);
 tft.fillScreen(TFT_RED);
 tft.setSwapBytes(true); // We need to swap the colour bytes (endianess)

 // The jpeg image can be scaled by a factor of 1, 2, 4, or 8
 TJpgDec.setJpgScale(1);

 // The decoder must be given the exact name of the rendering function above
 TJpgDec.setCallback(tft_output);
 Serial.println();
 Serial.println("Setting AP...");
 WiFi.softAP(ssid, password);

 IPAddress IP = WiFi.softAPIP();
 Serial.print("AP IP Address : ");
 Serial.println(IP);

 server.listen(80);
}

void loop() {
 if(server.poll()){
     client = server.accept();
   }

   if(client.available()){
     client.poll();

     WebsocketsMessage msg = client.readBlocking();
     Serial.printf("Received = %d bytes\n", msg.length());
     uint32_t t = millis();

     // Get the width and height in pixels of the jpeg if you wish
     uint16_t w = 0, h = 0;
     TJpgDec.getJpgSize(&w, &h, (const uint8_t*)msg.c_str(), msg.length());
     Serial.print("Width = "); Serial.print(w); Serial.print(", height = "); Serial.println(h);
  
     // Draw the image, top left at 0,0
     TJpgDec.drawJpg(0, 0, (const uint8_t*)msg.c_str(), msg.length());
  
     // How much time did rendering take (ESP8266 80MHz 271ms, 160MHz 157ms, ESP32 SPI 120ms, 8bit parallel 105ms
     t = millis() - t;
     Serial.print(t); Serial.println(" ms");
   } 
}
```

# Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Car Chassis Kit | Wheels and Base | $39.99 | <a href="https://www.amazon.com/dp/B0DJ7BT1V5?ref=cm_sw_r_cso_cp_apin_dp_RCSYWRX92M0H5DJ6HNQA&ref_=cm_sw_r_cso_cp_apin_dp_RCSYWRX92M0H5DJ6HNQA&social_share=cm_sw_r_cso_cp_apin_dp_RCSYWRX92M0H5DJ6HNQA&rsd=oU5zjHTwUjufNpZkC6CW0sqRlEipy6Xgf59f5777Kxh7cknbp6DwTNVEgVR1R1%2FY0I8OXRT9EOeWKVF0ff4yEbtnF%2Fc9MNo6yf5KfYW6Lx%2BkqE4%3D&edk=AQIDAHi1lw%2FM8UbbSMD9ScOOFEmBMHMthHeEhqDaQYPJUAX3jQHYb0B2nFfwd4jzBFZyiYMUAAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQM7ULhz148q%2B1PjBJVAgEQgDvE8maRGRFUIB7tnUdXxocbXxxr5gXUvho7mquZi7Zok3ViYk7wwVFTYIEajFhVByN74efn2RX1qaf%2BHQ%3D%3D"> Link </a> |
| Screwdriver Kit | Putting together parts | $5.94 | <a href="[https://www.amazon.com/Small-Screwdriver-Set-Mini-Magnetic/dp/B08RYXKJW9/](https://www.amazon.com/Small-Screwdriver-Set-Mini-Magnetic/dp/B08RYXKJW9/)"> Link </a> |
| Arduino Uno Clone | What the item is used for | $14.98 | <a href="https://www.amazon.com/ELEGOO-Board-ATmega328P-ATMEGA16U2-Compliant/dp/B01EWOE0UU/ref=sr_1_2_sspa?crid=3A6NCD2X9JEMJ&dib=eyJ2IjoiMSJ9.AcWZy-Yg4mDTnhzEHozxzPZdVC5-KUL2tW-OQewDKpBB4brSpD-p4bn74WcXiW3KarYertgpNaLJ0VHKx0qsPqolKAhiz1GRG5BwJQl73cEvrlXIXNmqlpSvU7uu2aRVSwAZi9Gj2AjSPLM3esW1Gzy9xEiQ9oiR5LCNjh4MlYDx5mTm5sI4rsD4CFTipJnF572qXlickl35FRcCj8oMXQotumgqI4yEIq0HobOtIlEnNhtVB51JMBHhqtmmF_PC9WeHJ4ySUVVcv_gq3_VeG1aAEbdm4NXmmT6NOYPw4Qo.1PFdgFT22oqO5Mg6-6j_aUL_EV8tUPuaFrB5N9oaEX0&dib_tag=se&keywords=elegoo+arduino&qid=1716856465&s=electronics&sprefix=elegoo+arduino%2Celectronics%2C99&sr=1-2-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Electronics Kit | Tools for electronics | $14 | <a href="https://www.amazon.com/Smraza-Electronics-Potentiometer-tie-Points-Breadboard/dp/B0B62RL725/ref=sxts_b2b_sx_reorder_acb_business?content-id=amzn1.sym.f63a3b0b-3a29-4a8e-8430-073528fe007f%3Aamzn1.sym.f63a3b0b-3a29-4a8e-8430-073528fe007f&crid=2IC3T44H3U3WG&cv_ct_cx=breadboard+kit&dib=eyJ2IjoiMSJ9.TUd5tu2T8rmms7ZuJ0UzmbtpLL1zsu93bQM0PzwnP4E.sT0V0vL_QtbYv8ymVTCcRkhFNgBtRvRiT7G4FT1oGTE&dib_tag=se&keywords=breadboard+kit&pd_rd_i=B0B62RL725&pd_rd_r=67e1f4ff-e3b9-44e4-b441-b4ae282f036b&pd_rd_w=UjFaP&pd_rd_wg=0xRoC&pf_rd_p=f63a3b0b-3a29-4a8e-8430-073528fe007f&pf_rd_r=BFGP77H27ZN31W4PZAW6&qid=1715911733&sbo=RZvfv%2F%2FHxDF%2BO5021pAnSA%3D%3D&sprefix=breadboard+kit%2Caps%2C109&sr=1-2-9f062ed5-8905-4cb9-ad7c-6ce62808241a"> Link </a> |
| Breadboard Kit | Connecting electronic parts | $8.79 | <a href="https://www.amazon.com/Breadboards-Solderless-Breadboard-Distribution-Connecting/dp/B07DL13RZH/ref=sxts_b2b_sx_reorder_acb_business?content-id=amzn1.sym.f63a3b0b-3a29-4a8e-8430-073528fe007f%3Aamzn1.sym.f63a3b0b-3a29-4a8e-8430-073528fe007f&crid=1RAL6PA1TZ81Q&cv_ct_cx=breadboard+kit&dib=eyJ2IjoiMSJ9.TUd5tu2T8rmms7ZuJ0UzmbtpLL1zsu93bQM0PzwnP4E.sT0V0vL_QtbYv8ymVTCcRkhFNgBtRvRiT7G4FT1oGTE&dib_tag=se&keywords=breadboard+kit&pd_rd_i=B07DL13RZH&pd_rd_r=1e3e6f57-5578-4452-b230-90d43c79b5d3&pd_rd_w=rFN6B&pd_rd_wg=3mMuA&pf_rd_p=f63a3b0b-3a29-4a8e-8430-073528fe007f&pf_rd_r=JC9D7T4VYRDQ9HJVY5X8&qid=1715912837&s=electronics&sbo=RZvfv%2F%2FHxDF%2BO5021pAnSA%3D%3D&sprefix=breadboard+kit%2Celectronics%2C102&sr=1-1-9f062ed5-8905-4cb9-ad7c-6ce62808241a"> Link </a> |
| Arduino Nano 33 BLE Sense | Bluetooth Sensor | $39.7 | <a href="https://www.amazon.com/Arduino-Nano-Sense-headers-ABX00070/dp/B0BQHZ88WD/ref=sr_1_4?crid=1BTYPUQCTIWYN&dib=eyJ2IjoiMSJ9.5ykyUyT10Vdnbme1Ur85NoPh9YmzeyxQKWTP0jF0ju7Zw9b2hLtWjY3pTyREGe5HkneZz75CgR3J9S8HJbMwmvkj1c1Mu9x0rZ651S1aBHwNqxIYbKjWG8yzYzDh5tcKP57E9RxRmqavQMCJ-QtCLIFas8oQKdBZDx67b_JUYJ3hdfDjHDXrimHAEzVTZrVAwh6NOXZ8-yMZIcp72LVtDsuQxyCvkrDyZM1EbuZQHlc.iy6QHwMR4-UrQZrInFc0eTSZJP6LewrRVqwpOfrQCG0&dib_tag=se&keywords=arduino+nano+33+ble&qid=1748096993&sprefix=arduino+nano+33ble%2Caps%2C151&sr=8-4"> Link </a> |
| Mirco USB Cable | Connecting parts | $5 | <a href="https://www.amazon.com/Charging-Transfer-Android-Trustable-MYFON/dp/B098DW7485/ref=sr_1_6?crid=3USJU0DMSZB2S&keywords=micro+usb&qid=1686187078&s=electronics&sprefix=micro+usb%2Celectronics%2C106&sr=1-6"> Link </a> |
| Accelerometer | Measure Acceleration | $9 | <a href="https://www.amazon.com/dp/B0D2TJVMNY?ref=fed_asin_title"> Link </a> |
| HC05 | Bluetooth button | $9 | <a href="https://www.amazon.com/DSD-TECH-HC-05-Pass-through-Communication/dp/B01G9KSAF6/ref=sr_1_3?crid=2J833J7AYQJA&keywords=hc05&qid=1686187263&sprefix=hc0%2Caps%2C112&sr=8-3"> Link </a> |
| Breadboard Power Supply | Output power  | $8 | <a href="https://www.amazon.com/ALAMSCN-Solderless-Breadboard-Battery-Arduino/dp/B08JYPMCZY/ref=sxts_b2b_sx_reorder_acb_business?content-id=amzn1.sym.f63a3b0b-3a29-4a8e-8430-073528fe007f%3Aamzn1.sym.f63a3b0b-3a29-4a8e-8430-073528fe007f&crid=Z2S8NZU0KN1S&cv_ct_cx=breadboard+power+supply&dib=eyJ2IjoiMSJ9.nJ_euybTOUu9E6yyDpnEqg.NgztCYPGkG96eXyyFxpvxOVw5ykdTUq6oziUQnvf51E&dib_tag=se&keywords=breadboard+power+supply&pd_rd_i=B08JYPMCZY&pd_rd_r=f2beb6df-6d77-44a3-8b72-83255f19ca20&pd_rd_w=r1wmq&pd_rd_wg=ToFNq&pf_rd_p=f63a3b0b-3a29-4a8e-8430-073528fe007f&pf_rd_r=R5ZMMGW4CXRBP3PWAYMA&qid=1715912515&s=electronics&sbo=RZvfv%2F%2FHxDF%2BO5021pAnSA%3D%3D&sprefix=breadboard+power+s%2Celectronics%2C114&sr=1-1-9f062ed5-8905-4cb9-ad7c-6ce62808241a"> Link </a> |
| 9v Batteries | Store and release energy | $8.69 | <a href="https://www.amazon.com/Amazon-Basics-Performance-All-Purpose-Batteries/dp/B00MH4QM1S/ref=sr_1_5_pp?crid=3TQ7ANPH958JM&dib=eyJ2IjoiMSJ9.bmcV2Upj_vpB6G9CFlPPxYAryat512da7ekZjc52HecXSTmtx7PbJ50EgQFPCMqlAxjOUq-tL4vQTpozlHvH89bMwx-HJoyGcdz6EY8HrMxahTiqOXkoP7ewkDcgHoMhmHamdlQfW6FBHO0Gm-DYZZnnMuvEU3qOpemA8PGEvRhEx4-lGaBZhrvls039G1-9SizAW-YRGXZ2fFrdVDlREyyOhAuxXZaE5QqUxWesRQgP9UfGOYaInRWTTPwhDbXFa-RPzGbU1C_u4wq-NMqKBtWEQqR9-cA8O3FYOx3icEY.dtKJmI2T-iCmMM_bYnbiHUWzhKpJDRxS-bBmZIwYFKM&dib_tag=se&keywords=9v+batteries&qid=1720651326&rdc=1&s=electronics&sprefix=9v+batteries%2Celectronics%2C105&sr=1-5"> Link </a> |
| Velcro Tape | Strapping and securing | $8 | <a href="https://www.amazon.com/Art3d-Sticky-Double-Sided-Command-Adhesive/dp/B0B58FGF8H/ref=sr_1_1_sspa?crid=2N0JOMEZLJ2DS&dib=eyJ2IjoiMSJ9.qGUGB_MXfmbL0MW7bqNJbxvZC9pzliDJ9KYyRNNrctnh03kCcUXONRrcPYdGeo7Jwzrm83HyF8Jsb1RkcdlLPAw-8RkxbTCMiW6UI1Fpnjv9GjXUg9VBOLxmLVUbmMp5J7gFXKKLTWQ-w_L4Q9rykEUqKmjv-v6GRykMMZLY2cVt__lLxMIlwr6qBnQLWpHiklifUJwjiURxO--TTt2VReYgmN0z7118ifSucrkvRrg.mwA0L4zMSlJP2RO8IBba7dVqwa1Lkr8KvY1JmeQEfCg&dib_tag=se&keywords=velcro+tape+pieces&qid=1716734034&sprefix=velcro+tape+piece%2Caps%2C89&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| DMM | Measures Voltage | $9.99 | <a href="https://www.amazon.com/dp/B0CXM242J1?ref=fed_asin_title&th=1"> Link </a> |

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
