#include <WiFi.h>
#include <WebServer.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <time.h>

const char* ssid = "OJI"; //SSID WIFI yang digunakan
const char* password = "OnejectOJI88!"; //Passsword WIFI 
const char* scriptURL = "https://script.google.com/macros/s/AKfycbwVFrP5pP18-7c_LFjP_pqilvWdEAkJ4rLQRrOwwP4Hp_kgN0dRqTads4bqfNdSx2onWA/exec";

const int muxS0 = 18;
const int muxS1 = 26;
const int muxS2 = 27;

const int muxEN1 = 14;
const int muxEN2 = 12;
const int muxEN3 = 13;

const int muxSIG = 34;
const int totalChannels = 4; //Jumlah jenis nepel yang dideteksi

WebServer server(80);

int lastSensorValues[totalChannels];
unsigned long falseStartTimes[totalChannels];
unsigned long lastCheckTime = 0;
const unsigned long checkInterval = 1000;

const char* namaNepel[totalChannels] = {"Nepel 12-12", "Nepel 12-10", "Nepel 10-10", "Nepel Single 10"}; //Daftar nama nepel yang dideteksi (diurutkan dari channel pertama)
String logs[10];
int logIndex = 0;

bool internetStatus = false;
bool googleSheetStatus = false;
int googleSheetCode = -1;
String googleSheetMessage = "Not Sent";

bool statusChanged = false;
bool wifiReconnected = false;

String lastUpdateTimestamps[totalChannels];

unsigned long lastWifiCheckTime = 0;
unsigned long wifiConnectedTime = 0;

void setup() {
  Serial.begin(115200);
  pinMode(muxS0, OUTPUT);
  pinMode(muxS1, OUTPUT);
  pinMode(muxS2, OUTPUT);

  pinMode(muxEN1, OUTPUT);
  digitalWrite(muxEN1, LOW);
  digitalWrite(muxEN2, HIGH);
  digitalWrite(muxEN3, HIGH);
  pinMode(muxSIG, INPUT);

  connectToWiFi();

  configTime(0, 0, "pool.ntp.org", "time.nist.gov");
  waitForNTP();
  
  for (int ch = 0; ch < totalChannels; ch++) {
    lastSensorValues[ch] = readMUX(ch);
    lastUpdateTimestamps[ch] = getLocalTimeString();
  }

  server.on("/", handleRoot);
  server.on("/changed", []() {
    if (statusChanged || wifiReconnected) {
      statusChanged = false;
      wifiReconnected = false;
      server.send(200, "text/plain", "changed");
    } else {
      server.send(200, "text/plain", "nochange");
    }
  });
  server.begin();
}

void loop() {
  server.handleClient();
  if (millis() - lastWifiCheckTime >= 10000) {
    lastWifiCheckTime = millis();
    if (WiFi.status() != WL_CONNECTED) {
      Serial.println("WiFi Disconnected! Attempting reconnect...");
      connectToWiFi();
      if (WiFi.status() == WL_CONNECTED) {
        wifiReconnected = true;
        Serial.println("WiFi Reconnected!");
      }
    }
  }

  internetStatus = isInternetConnected();
  if (millis() - lastCheckTime >= checkInterval) {
    lastCheckTime = millis();
    for (int ch = 0; ch < totalChannels; ch++) {
      int currentValue = readMUX(ch);
      if (currentValue != lastSensorValues[ch]) {
        statusChanged = true;
        String nama = namaNepel[ch];
        String status = currentValue == 1 ? "runout" : "ready";
        unsigned long duration = 0;
        if (currentValue == 0) falseStartTimes[ch] = millis();
        else if (lastSensorValues[ch] == 0)
          duration = millis() - falseStartTimes[ch];

        lastSensorValues[ch] = currentValue;
        lastUpdateTimestamps[ch] = getLocalTimeString();
        sendStatusToGoogleSheet(nama, status);

        String logEntry = getLocalTimeString() + " - " + nama + " menjadi " + status;
        logs[logIndex % 10] = logEntry;
        logIndex++;
      }
    }
  }
}

void connectToWiFi() {
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  unsigned long startAttemptTime = millis();
  while (WiFi.status() != WL_CONNECTED && millis() - startAttemptTime < 10000) {
    Serial.print(".");
    delay(500);
  }
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\\nWiFi Connected: " + WiFi.localIP().toString());
    sendIPToGoogleSheet(WiFi.localIP().toString());
    wifiConnectedTime = millis();
  } else {
    Serial.println("\\nWiFi Connection Failed.");
  }
}

int readMUX(int channel) {
  digitalWrite(muxS0, bitRead(channel, 0));
  digitalWrite(muxS1, bitRead(channel, 1));
  digitalWrite(muxS2, bitRead(channel, 2));
  delay(20);
  return digitalRead(muxSIG);
}

void waitForNTP() {
  Serial.print("Waiting for NTP time sync");
  time_t now = time(nullptr);
  int attempts = 0;

  while (now < 8 * 3600 * 2 && attempts < 20) { // tunggu hingga waktu valid (lebih dari 1970)
    delay(500);
    Serial.print(".");
    now = time(nullptr);
    attempts++;
  }

  if (now >= 8 * 3600 * 2) {
    Serial.println("\nNTP time synchronized.");
  } else {
    Serial.println("\nNTP sync failed, using fallback time.");
  }
}

void handleRoot() {
  bool isTimestampValid = true;
  for (int i = 0; i < totalChannels; i++) {
    if (lastUpdateTimestamps[i].startsWith("1970")) {
      isTimestampValid = false;
      break;
    }
  }

  String html = "<!DOCTYPE html><html lang='en'><head><meta charset='UTF-8' />";
  html += "<meta name='viewport' content='width=device-width, initial-scale=1.0' />";
  if (!isTimestampValid) {
    html += "<meta http-equiv='refresh' content='2'>";  // Tambahkan ini jika timestamp belum valid
  }
  html += "<title>DASHBOARD MONITORING STOCK NEPEL</title>";
  html += "<style>body{margin:0;font-family:Arial,sans-serif;background:#f8f8f8}";
  html += "header{background-color:#4b9cdb;color:white;padding:20px 40px;box-shadow:0 2px 5px rgba(0,0,0,0.1)}";
  html += "header h1{margin:0;font-size:28px;font-weight:bold;text-shadow:2px 2px 0px #2c72a8}";
  html += "header h2{margin:5px 0 0 0;font-size:16px;font-weight:600}";
  html += ".container{display:flex;flex-wrap:wrap;gap:30px;padding:20px 40px}";
  html += ".status-section{flex:1 1 400px;max-width:700px}";
  html += ".log-section{flex:1 1 300px;max-width:600px}";
  html += ".status-section h3{text-align:center;margin-bottom:15px}";
  html += ".status-cards{display:flex;gap:15px;flex-wrap:wrap;justify-content:center;background:white;padding:20px;border-radius:20px;box-shadow:6px 6px 0px #aaa}";
  html += ".card{color:white;padding:15px;border-radius:20px;width:120px;text-align:center;font-weight:bold}";
  html += ".ready{background:#2b7a1f;}.runout{background:#c0392b;}.card small{display:block;font-weight:normal;font-size:12px;margin-top:5px}";
  html += ".uptime-box{margin-top:20px;background:white;padding:20px;border-radius:20px;text-align:center;box-shadow:4px 4px 8px rgba(0,0,0,0.1)}";
  html += ".uptime-box .uptime-title{font-size:16px;font-weight:bold;margin-bottom:8px;color:#444}";
  html += ".uptime-box #uptime{font-size:20px;color:#2b7a1f;font-weight:bold}";
  html += ".log-table{background:white;border-radius:20px;overflow:hidden;box-shadow:3px 3px 10px rgba(0,0,0,0.1)}";
  html += ".log-header{background:#4b9cdb;color:white;padding:10px;font-weight:bold;display:flex;justify-content:space-between;text-align:center}";
  html += ".log-header div{flex:1;padding:10px;text-align:center;min-width:0;box-sizing:border-box}";
  html += ".log-body{padding:10px 0;font-size:14px}.log-row{display:flex;justify-content:space-between;margin-bottom:5px}";
  html += ".log-row div{flex:1;padding:10px;text-align:center;min-width:0;box-sizing:border-box}";
  html += "@media screen and (max-width: 767px){.container{flex-direction:column;padding:10px}.status-section,.log-section{max-width:100%;padding:0}";
  html += ".status-cards{flex-direction:column;align-items:center}.card{width:80%}.uptime-box{margin-top:20px}}";
  html += ".log-link{text-align:center;margin-top:15px}";
  html += ".log-button{display:inline-block;background:#4b9cdb;color:white;padding:12px 24px;border-radius:10px;font-weight:bold;font-size:16px;text-decoration:none;box-shadow:2px 2px 6px rgba(0,0,0,0.2);transition:0.2s ease-in-out}";
  html += ".log-button:hover{background:#357ab8;transform:translateY(-2px);cursor:pointer}";
  html += "</style></head><body>";
  html += "<header><h1>DASHBOARD MONITORING STOCK NEPEL</h1><h2>INJECTION MOLDING</h2></header>";
  html += "<div class='container'><div class='status-section'><h3>STATUS</h3><div class='status-cards'>";

  for (int ch = 0; ch < totalChannels; ch++) {
    int value = lastSensorValues[ch];
    String name = String(namaNepel[ch]);
    String statusClass = value == 1 ? "runout" : "ready";
    String timestamp = lastUpdateTimestamps[ch];
    String dateStr = timestamp.substring(0, 10);
    String timeStr = timestamp.substring(11);

    html += "<div class='card " + statusClass + "'>" + name;
    html += "<small>Date: " + dateStr.substring(8,10) + "/" + dateStr.substring(5,7) + "/" + dateStr.substring(0,4) + "</small>";
    html += "<small>Time: " + timeStr + "</small></div>";
  }

  html += "</div>"; // end .status-cards

  // Tambahkan kotak uptime di bawah status-cards
  html += "<div class='uptime-box'>";
  html += "<div class='uptime-title'>UPTIME</div>";
  html += "<div id='uptime'>" + getUptimeString() + "</div>";
  html += "</div>"; // end .uptime-box

  // Tombol link ke spreadsheet
  html += "<div class='log-link'>";
  html += "<a href='https://docs.google.com/spreadsheets/d/1QFN6b2QaQEETtJYHuudSkTxjHCEVqrDJlAq6RTz8lxI/edit?usp=sharing' target='_blank'>";
  html += "<div class='log-button'>&#10148; Full Logs</div></a></div>";

  html += "</div><div class='log-section'><div class='log-table'>";
  html += "<div class='log-header'><div>TIMESTAMP</div><div>NAME</div><div>STATUS</div></div><div class='log-body'>";

  int displayCount = min(logIndex, 10);

  for (int i = 0; i < displayCount; i++) {
    int index = (logIndex - 1 - i + 10) % 10;
    String log = logs[index];
    int sep1 = log.indexOf(" - ");
    int sep2 = log.indexOf(" menjadi ");
    String timestamp = log.substring(0, sep1);
    String name = log.substring(sep1 + 3, sep2).substring(6);
    String stat = log.substring(sep2 + 9);
    stat.replace("runout", "Runout");
    stat.replace("ready", "Ready");

    html += "<div class='log-row'><div>" + timestamp + "</div><div>" + name + "</div><div>" + stat + "</div></div>";
  }

  html += "</div></div></div></div>";
  html += "<script>";
  html += "setInterval(() => { fetch('/changed').then(res => res.text()).then(data => { if(data === 'changed'){ location.reload(); }}); }, 2000);";
  html += "setInterval(() => { const el = document.getElementById('uptime'); if(el){ let parts = el.textContent.split(':').map(Number); let h=parts[0],m=parts[1],s=parts[2]; s++; if(s>=60){s=0;m++;} if(m>=60){m=0;h++;} el.textContent=`${String(h).padStart(2,'0')}:${String(m).padStart(2,'0')}:${String(s).padStart(2,'0')}`; } }, 1000);";
  html += "</script></body></html>";

  server.send(200, "text/html", html);
}

void sendIPToGoogleSheet(String ip) {
  HTTPClient http;
  String encodedIP = "'" + ip;
  String url = String(scriptURL) + "?ip=" + encodedIP;
  Serial.println("Sending to: " + url);
  http.begin(url);
  int code = http.GET();
  Serial.println("Response code: " + String(code));
  http.end();
}

void sendStatusToGoogleSheet(String nama, String status) {
  HTTPClient http;
  String encodedNama = urlEncode(nama);
  String encodedStatus = urlEncode(status);
  String url = String(scriptURL) + "?nama=" + encodedNama + "&status=" + encodedStatus;
  http.begin(url);
  int httpCode = http.GET();
  googleSheetCode = httpCode;
  googleSheetStatus = (httpCode > 0 && httpCode < 400);
  googleSheetMessage = httpCode > 0 ? "OK" : "Error: " + String(http.errorToString(httpCode));
  http.end();
}

String getLocalTimeString() {
  time_t rawtime = time(nullptr) + 7 * 3600;
  struct tm* timeinfo = gmtime(&rawtime);
  char buffer[30];
  strftime(buffer, sizeof(buffer), "%Y-%m-%d %H:%M:%S", timeinfo);
  return String(buffer);
}

String getUptimeString() {
  unsigned long seconds = (millis() - wifiConnectedTime) / 1000;
  unsigned int h = seconds / 3600;
  unsigned int m = (seconds % 3600) / 60;
  unsigned int s = seconds % 60;
  char buffer[10];
  sprintf(buffer, "%02u:%02u:%02u", h, m, s);
  return String(buffer);
}


bool isInternetConnected() {
  WiFiClient testClient;
  return testClient.connect("google.com", 80);
}

String urlEncode(const String& str) {
  String encoded = "";
  char c;
  char code[4];
  for (int i = 0; i < str.length(); i++) {
    c = str.charAt(i);
    if (isalnum(c)) {
      encoded += c;
    } else {
      sprintf(code, "%%%02X", c);
      encoded += code;
    }
  }
  return encoded;
}
