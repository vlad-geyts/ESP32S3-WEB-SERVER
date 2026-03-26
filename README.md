Phase 5.1: The C++17 Web Server

Now that you have an IP address (192.168.1.18), let's make it useful. We will create a simple Web Server 
so you can view the panic_count from your phone or PC.

C++17 Goal: We will use std::string and std::to_string to build our HTML response dynamically.

Step 1: Add the Library
You'll need the WebServer library. It's built into the ESP32 core. Add #include <WebServer.h> to your headers.

Step 2: Define the Server
Add this globally:
                    WebServer server(80); // Create a server on port 80 (standard HTTP)

Step 3: The "Handle Root" Logic (C++17 style)
Add this function to build the page:
                void handleRoot() {
                    prefs.begin("system", true);
                    auto count = prefs.getUInt("panic_count", 0);
                    prefs.end();

                    // Use std::string for easy concatenation
                    std::string html = "<html><body><h1>ESP32-S3 Panic Monitor</h1>";
                    html += "<p>Lifetime Events: <b>" + std::to_string(count) + "</b></p>";
                    html += "<p>WiFi Signal: " + std::to_string(WiFi.RSSI()) + " dBm</p>";
                    html += "</body></html>";

                    server.send(200, "text/html", html.c_str());
                }

Step 4: Update setup()
Add these lines before vTaskDelete(NULL):
                server.on("/", handleRoot);
                server.begin();
                Serial.println("HTTP Server Started.");

Step 5: The "Server Task"
The server needs to be "ticked" constantly to listen for clients. Since loop() is dead, 
we need a small task for this.
                void serverTask(void* pvParameters) {
                    for(;;) {
                    server.handleClient();
                    vTaskDelay(pdMS_TO_TICKS(5)); // Small delay to let other tasks breathe
                    }
                }   

Let’s integrate that web server. Since you are moving toward C++17, we’ll use a few more modern 
features like std::string for the HTML generation and auto for type deduction.

One important detail: We will create a dedicated task for the Web Server. This keeps the network
 handling separate from your high-priority panicTask.
 
Why we did this:
1.  serverTask: By putting server.handleClient() in its own task on Core 0, your web page 
    will load even while the panicTask is busy strobing LEDs on Core 1. 
2.  std::to_string(count): This is a very handy C++ feature. It converts numbers to strings 
    automatically, so you don't have to use messy sprintf buffers.    
3.  The Meta-Refresh: I added <meta http-equiv='refresh' content='2'> to the HTML. 
    This tells your browser to auto-reload the page every 2 seconds, so you can watch 
    the panic count increment in real-time without clicking refresh!

How to test: 1. Flash the code.
2. Open your browser on your phone/PC.
3. Enter the IP address shown in your Serial Monitor (e.g., http://192.168.1.18).
4. Press your physical button and watch the webpage update!

Building a web server is a big step—it's the point where your embedded hardware finally starts "talking" to the rest of the world.

When you get it running, try this little experiment:
 1. Open the page on your smartphone.
 2. Press the physical button.
 3. Observe how the count updates on your phone screen within 2 seconds (thanks to that meta-refresh tag).

If you run into any compiler errors with std::string or the WebServer library, just paste them here and we'll troubleshoot them.