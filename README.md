# ESP32 WhatsApp Messaging Project

For this project, I used an ESP32-CAM to send a WhatsApp message to my phone using the CallMeBot API. 

---

## How It Works

The ESP32 connects to your Wi-Fi network, and then sends a custom WhatsApp message to your phone using a link that includes your phone number, the message itself, and an API key. 
---


## Setup Instructions

1. **Add CallMeBot to WhatsApp:**
   - Save the number `+34 698 28 89 73` to your contacts
   - Open WhatsApp and message it:
     ```
     I allow callmebot to send me messages
     ```
   - It will reply with your personal API key

2. **Set up your ESP32 in Arduino IDE:**
   - Select the right board: `AI Thinker ESP32-CAM`
   - Choose your COM port
   - Make sure baud rate in Serial Monitor is set to `115200`

3. **Upload the code:**
   - Replace the Wi-Fi name and password, your WhatsApp number (with country code), and your CallMeBot API key
   - Upload the sketch and open Serial Monitor
   - You should see “WiFi connected” and “Message sent successfully”

---

## Troubleshooting (What Didn't Work At First)

At first, the ESP32 said the message was “sent,” but I got a **400 Bad Request** error from the server. Here’s what I had to fix:

- I had typed the wrong API server URL. It should be:  
  `https://api.callmebot.com`, **not** `api2.callmebot.com`
- I wasn’t encoding the message text properly. Even simple punctuation (like commas and spaces) caused issues.
- I added a function to encode the message using URL-safe formatting
- I double-checked my phone number format (`+1XXXXXXXXXX`) and copied the API key exactly from WhatsApp

Once I made these changes, the message went through perfectly.

---

## Final Result

The ESP32 connected to Wi-Fi and sent this message to my WhatsApp:
> *“Hello, from ESP32!”*

Seeing it show up on my phone in real-time was the best part of the whole project. 

















#include <WiFi.h>
#include <HTTPClient.h>

const char* ssid = "Dreamk2";          
const char* password = "xx xxxxx xxxx";  

String phoneNumber = "+19173651310";   
String apiKey = "5038236";            
String message = "Hello, from ESP32!";

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);

  Serial.print("Connecting to WiFi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi connected.");
  Serial.println("Sending WhatsApp message...");

  sendWhatsAppMessage(message);
}

void loop() {

}

void sendWhatsAppMessage(String msg) {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    String encodedMessage = urlencode(msg);
    String url = "https://api.callmebot.com/whatsapp.php?phone=" + phoneNumber + "&text=" + encodedMessage + "&apikey=" + apiKey;

    http.begin(url);
    int httpCode = http.GET();

    if (httpCode > 0) {
      Serial.println("Message sent successfully.");
      Serial.println(http.getString());
    } else {
      Serial.print("Error sending message. HTTP code: ");
      Serial.println(httpCode);
    }

    http.end();
  } else {
    Serial.println("WiFi not connected.");
  }
}


String urlencode(String str) {
  String encodedString = "";
  char c;
  char code0;
  char code1;
  for (int i = 0; i < str.length(); i++) {
    c = str.charAt(i);
    if (isalnum(c)) {
      encodedString += c;
    } else {
      code1 = (c & 0xf) + '0';
      if ((c & 0xf) > 9) code1 = (c & 0xf) - 10 + 'A';
      code0 = ((c >> 4) & 0xf) + '0';
      if (((c >> 4) & 0xf) > 9) code0 = ((c >> 4) & 0xf) - 10 + 'A';
      encodedString += '%';
      encodedString += code0;
      encodedString += code1;
    }
  }
  return encodedString;
}






 
