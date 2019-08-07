/*************************************************************
  Download latest Blynk library here:
    https://github.com/blynkkk/blynk-library/releases/latest

  Blynk is a platform with iOS and Android apps to control
  Arduino, Raspberry Pi and the likes over the Internet.
  You can easily build graphic interfaces for all your
  projects by simply dragging and dropping widgets.

    Downloads, docs, tutorials: http://www.blynk.cc
    Sketch generator:           http://examples.blynk.cc
    Blynk community:            http://community.blynk.cc
    Follow us:                  http://www.fb.com/blynkapp
                                http://twitter.com/blynk_app

  Blynk library is licensed under MIT license
  This example code is in public domain.

 *************************************************************
  This example runs directly on NodeMCU.

  Note: This requires ESP8266 support package:
    https://github.com/esp8266/Arduino

  Please be sure to select the right NodeMCU module
  in the Tools -> Board menu!

  For advanced settings please follow ESP examples :
   - ESP8266_Standalone_Manual_IP.ino
   - ESP8266_Standalone_SmartConfig.ino
   - ESP8266_Standalone_SSL.ino

  Change WiFi ssid, pass, and Blynk auth token to run :)
  Feel free to apply it to any other example. It's simple!
 *************************************************************/

/* Comment this out to disable prints and save space */
#define BLYNK_PRINT Serial
#include <EEPROM.h>
#include <FastLED.h>
#define DATA_PIN    5
//#define CLK_PIN   4
#define LED_TYPE    WS2811
#define COLOR_ORDER GRB
#define NUM_LEDS    10
CRGB leds[NUM_LEDS];
int EEaddress = 0, EEaddress2 = 150;
#define BRIGHTNESS          96
#define FRAMES_PER_SECOND  120
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>

// You should get Auth Token in the Blynk App.
// Go to the Project Settings (nut icon).
char auth[] = "3eb4b6881ecf45499d372ac29e772b61";
int RainbowColors_Switch, Brightness = 200, Red = 164, Green = 38, Blue = 0, Connection_Waiting_Count = 0;
int Fade_Switch = 0, Flag_1 = 0, ChangeInBrightness = 0, Password_Change_Authorisation;
float Rate_Of_Fade = 3, Temp_Value;
int Step = 0;
String  Correction_Conformation;
// Your WiFi credentials.
// Set password to "" for open networks.
char Temporary_String_1[100], Temporary_String_2[100];
String ssid_WiFi ;
String Password_WiFi ;
String ssid_HotSpot     = "node";
String Password_HotSpot = "12345678";
BlynkTimer timer;
WidgetTerminal terminal(V8);
BLYNK_WRITE(V0) // Widget WRITEs to Virtual Pin V1
{
  Red = param.asInt(); // getting first value
}
BLYNK_WRITE(V1) // Widget WRITEs to Virtual Pin V1
{
  Green = param.asInt(); // getting first valu
}
BLYNK_WRITE(V2) // Widget WRITEs to Virtual Pin V1
{
  Blue = param.asInt(); // getting first value
}
BLYNK_WRITE(V3) // Widget WRITEs to Virtual Pin V1
{
  Brightness = param.asInt(); // getting first value
}
BLYNK_WRITE(V4) // Widget WRITEs to Virtual Pin V1
{
  RainbowColors_Switch = param.asInt(); // getting first value
}
BLYNK_WRITE(V5) // Widget WRITEs to Virtual Pin V1
{
  Fade_Switch = param.asInt(); // getting first value
}
BLYNK_WRITE(V6) // Widget WRITEs to Virtual Pin V1
{
  Rate_Of_Fade = param.asInt(); // getting first value
}
BLYNK_WRITE(V7) // Widget WRITEs to Virtual Pin V1
{
  Password_Change_Authorisation = param.asInt(); // getting first value
}

BLYNK_WRITE(V8) // Widget WRITEs to Virtual Pin V1
{
  if (Password_Change_Authorisation == 1)
  {
    if (Step == 1)
    {
      if (ssid_WiFi = param.asStr())
      {
        Step = 2;
      }
    }
    if (Step == 2)
    {
      terminal.println("say me ur wifi password") ;

      terminal.flush();
      Step = 3;
    }
    if (Step == 3)
    {
      if (ssid_WiFi != param.asStr())
      {
        Password_WiFi = param.asStr();
        Step = 4;
      }
    }
    if (Step == 4)
    {
      Serial.print("ssid_WiFi:");
      Serial.println(ssid_WiFi);
      Serial.print("pass:");
      Serial.println(Password_WiFi);
      terminal.print("wifi name:");
      terminal.println(ssid_WiFi);
      terminal.print("password :");
      terminal.println(Password_WiFi);
      terminal.println("if the entered data is correct type 'yes' else type 'no'");
      Step = 5;
    }
    if (Step == 5)
    {
      Correction_Conformation = param.asStr();
    }
    if (Correction_Conformation == "yes" )
    {
      /*for (int i = 0 ; i < EEPROM.length() ; i++) {
        EEPROM.put(i, 0);
        }
        EEPROM.commit();
        EEPROM.end();*/

      strcpy(Temporary_String_1, ssid_WiFi.c_str());
      strcpy(Temporary_String_2, Password_WiFi.c_str());
      EEPROM.put(0, Temporary_String_1);
      EEPROM.put(150, Temporary_String_2);
      EEPROM.commit();
      Serial.println(EEPROM.get(EEaddress, Temporary_String_1));
      Serial.println(EEPROM.get(EEaddress2, Temporary_String_2));
      terminal.println(F("password changed successfully"));
      Step = 0;
      Correction_Conformation = "";

    }
    if (Correction_Conformation == "no")
    {
      terminal.clear();
      Step = 1;
      terminal.println(F("say me ur wifi name")) ;
      Correction_Conformation = "";

    }
    // Ensure everything is sent
    terminal.flush();
  }
}

void show()
{
  Serial.print("RED:");
  Serial.println(Red);
  Serial.print("GREEN:");
  Serial.println(Green);
  Serial.print("BLUE:");
  Serial.println(Blue);
  Serial.print("BRIGHTNESS:");
  Serial.println(Brightness);
  //delay(500);
}
void   blinkled();



void setup()
{
  // Debug console
  Serial.begin(9600);
  EEPROM.begin(512);
  EEPROM.get(EEaddress, Temporary_String_1);
  EEPROM.get(EEaddress2, Temporary_String_2);
  ssid_WiFi = Temporary_String_1;
  Password_WiFi = Temporary_String_2;

  // tell FastLED about the LED strip configuration
  FastLED.addLeds<LED_TYPE, DATA_PIN, COLOR_ORDER>(leds, NUM_LEDS).setCorrection(TypicalLEDStrip);
  //FastLED.addLeds<LED_TYPE,DATA_PIN,CLK_PIN,COLOR_ORDER>(leds, NUM_LEDS).setCorrection(TypicalLEDStrip);
  for (int j = 0; j < NUM_LEDS; j++)
    leds[j] = CRGB(Blue, Red, Green);
  FastLED.show();
  // set master brightness control
  FastLED.setBrightness(BRIGHTNESS);
  Serial.print("Connecting to ");
  Serial.println(EEPROM.get(EEaddress, Temporary_String_1));


  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid_WiFi, Password_WiFi);

  while (WiFi.status() != WL_CONNECTED && Connection_Waiting_Count < 40) {
    delay(500);
    Serial.print(".");
    Connection_Waiting_Count++;
  }

  if (WiFi.status() != WL_CONNECTED)
  {
    Serial.print("Not connected");
    WiFi.begin(ssid_HotSpot, Password_HotSpot);

    while (WiFi.status() != WL_CONNECTED) {
      delay(500);
      Serial.print("*");
    }
  }
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  Blynk.config(auth);
  delay(1000);
  Blynk.connect();
  Blynk.virtualWrite(V0, Red);
  Blynk.virtualWrite(V1, Green);
  Blynk.virtualWrite(V2, Blue);
  Blynk.virtualWrite(V3, Brightness);
  Blynk.virtualWrite(V4, RainbowColors_Switch);
  Blynk.virtualWrite(V5, Fade_Switch);
  Blynk.virtualWrite(V6, Rate_Of_Fade);
  Blynk.virtualWrite(V7, Password_Change_Authorisation);
  terminal.flush();
  terminal.clear();
  // You can also specify server:
  //Blynk.begin(auth, ssid, pass, "blynk-cloud.com", 80);
  //Blynk.begin(auth, ssid, pass, IPAddress(192,168,1,100), 8080);
}

// List of patterns to cycle through.  Each is defined as a separate function below.
typedef void (*SimplePatternList[])();
//SimplePatternList gPatterns = { rainbow, rainbowWithGlitter, confetti, sinelon, juggle, bpm };

uint8_t gCurrentPatternNumber = 0; // Index number of which pattern is current
float gHue = 0; // rotating "base color" used by many of the patterns


void loop()
{

  Blynk.run();
  if (Step == 0 && Password_Change_Authorisation == 1)
  {
    delay(2000);
    terminal.clear();
    terminal.println(F("say me ur wifi name")) ;
    delay(20);
    terminal.flush();
    Step = 1;
  }

  if (Password_Change_Authorisation == 0)
  {
    terminal.clear();
    for (int j = 0; j < NUM_LEDS; j++)
      leds[j] = CRGB(Blue, Red, Green);
    if (RainbowColors_Switch == 1)
    {
      fill_rainbow( leds, NUM_LEDS, gHue, 7);


      if (Fade_Switch != 1)
      {
        gHue++;
        delay(180);
      }
      else
      {
        Temp_Value = Rate_Of_Fade / 50;
        Temp_Value = constrain(Temp_Value, 0.16, 2);
        gHue = gHue + (Temp_Value);
        Serial.println(Temp_Value);
      }
    }
    if (Fade_Switch == 1)
      blinkled();

    FastLED.show();
    FastLED.setBrightness(Brightness);
    if (Password_Change_Authorisation == 0)
      show();
  }
}
#define ARRAY_SIZE(A) (sizeof(A) / sizeof((A)[0]))
void blinkled()
{
  if (Flag_1 == 0 && ChangeInBrightness < 253)
  {
    ChangeInBrightness = ChangeInBrightness + Rate_Of_Fade;
  }
  else if (ChangeInBrightness > 1)
  {
    ChangeInBrightness = ChangeInBrightness - Rate_Of_Fade;
    Flag_1 = 1;
  }
  else
  {
    Flag_1 = 0;

  }
  ChangeInBrightness = constrain(ChangeInBrightness, 1, 254);

  for (int j = 0; j < NUM_LEDS; j++)
    leds[j].fadeToBlackBy(ChangeInBrightness);
  delay(20);
  FastLED.show();
}
