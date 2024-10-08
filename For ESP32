#include <WiFi.h>
#include <WebServer.h>
#include <SPIFFS.h>

const char* ssid = "XXXXX"; // Replace X with your WiFi SSID
const char* password = "XXXXX";  // Replace X with your WiFi Password

WebServer server(80);

void setup() {
  Serial.begin(115200);
  
  // Initialize SPIFFS
  if (!SPIFFS.begin(true)) {
    Serial.println("An error occurred while mounting SPIFFS");
    return;
  }
  
  // Connect to Wi-Fi
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nConnected to WiFi");

  // Define routes
  server.on("/", handleRoot);
  server.on("/upload", HTTP_POST, handleUpload, handleFileUpload);
  server.on("/download", HTTP_GET, handleDownload);
  server.on("/delete", HTTP_GET, handleDelete);

  // Start server
  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();
}

void handleRoot() {
  String html = "<html><body>";
  html += "<h1>ESP32 Cloud Storage</h1>";
  
  html += "<form action='/upload' method='POST' enctype='multipart/form-data'>";
  html += "<input type='file' name='file'><input type='submit' value='Upload'>";
  html += "</form>";
  
  // List Files
  html += "<h2>Uploaded Files:</h2><ul>";
  listFiles(html);
  html += "</ul>";

  html += "</body></html>";
  server.send(200, "text/html", html);
}

void listFiles(String &html) {
  // Open directory and list files
  File root = SPIFFS.open("/");
  File file = root.openNextFile();
  
  while (file) {
    String fileName = file.name();
    html += "<li>" + fileName + 
            " <a href='/download?file=" + fileName + "'>Download</a> " +
            "<a href='/delete?file=" + fileName + "'>Delete</a></li>";
    file = root.openNextFile();
  }
  root.close();
}

void handleUpload() {
  server.send(200, "text/html", "File uploaded successfully!");
}

void handleFileUpload() {
  HTTPUpload& upload = server.upload();
  if (upload.status == UPLOAD_FILE_START) {
    String filename = "/" + upload.filename;
    File file = SPIFFS.open(filename, "w");
    if (!file) {
      Serial.println("Failed to open file for writing");
      return;
    }
  } else if (upload.status == UPLOAD_FILE_WRITE) {
    File file = SPIFFS.open("/" + upload.filename, "a");
    if (file) {
      file.write(upload.buf, upload.currentSize);
      file.close();
    }
  } else if (upload.status == UPLOAD_FILE_END) {
    Serial.printf("Upload finished: %s, %d bytes\n", upload.filename.c_str(), upload.totalSize);
    return;
  }
}

void handleDownload() {
  String fileName = server.arg("file");
  if (fileName.isEmpty()) {
    server.send(400, "text/plain", "File not specified");
    return;
  }

  File file = SPIFFS.open("/" + fileName, "r");
  if (!file) {
    server.send(404, "text/plain", "File not found");
    return;
  }

  // Set the Content-Disposition header to prompt download
  server.sendHeader("Content-Disposition", "attachment; filename=\"" + fileName + "\"");
  server.streamFile(file, "application/octet-stream");
  file.close();
}

void handleDelete() {
  String fileName = server.arg("file");
  if (fileName.isEmpty()) {
    server.send(400, "text/plain", "File not specified");
    return;
  }

  if (SPIFFS.remove("/" + fileName)) {
    server.send(200, "text/plain", "File deleted");
  } else {
    server.send(404, "text/plain", "File not found");
  }
}
