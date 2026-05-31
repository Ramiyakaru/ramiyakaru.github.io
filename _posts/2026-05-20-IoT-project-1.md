---
title: "Building an IoT Temperature Monitoring System"
date: 26-05-20 +1030
categories: [IoT, Linux, arduino]
tags: [IoT, arduino github, linux, beginner]
---

## Building an IoT Temperature Monitoring System Using ESP8266, MQTT, Node-RED, and RGB LEDs

The Internet of Things (IoT) has become one of the most exciting technologies in modern computing. From smart homes and industrial automation to healthcare monitoring and agriculture, IoT devices are everywhere around us. In this blog post, we are going to build our own beginner-friendly IoT project using an ESP8266 microcontroller, a Keyestudio temperature sensor, Node-RED, MQTT, and an RGB LED strip.

This project is designed for beginners who want to start experimenting with IoT technologies while also learning networking, wireless communication, and hardware interfacing.

By the end of this project, we will be able to:

* Read temperature values from a sensor
* Display readings inside the Arduino IDE Serial Monitor
* Send temperature data wirelessly over Wi-Fi using MQTT
* Visualize real-time readings using Node-RED gauges
* Change RGB LED colors according to temperature values

The project is divided into three sections:

1. Displaying readings in the Arduino IDE Serial Monitor
2. Displaying readings wirelessly in a Node-RED gauge
3. Configuring an LED to light up different colors according to temperature readings

## What is IoT?

The Internet of Things (IoT) refers to physical devices connected to the internet that can collect, send, and receive data. These devices usually include sensors, microcontrollers, and communication modules that allow them to interact with other systems without human intervention.

Examples of IoT devices include:

* Smart thermostats
* Smart security cameras
* Smart refrigerators
* Industrial monitoring systems
* Smart irrigation systems
* Fitness trackers
* Smart vehicles

IoT combines hardware, software, networking, and cloud technologies to create intelligent systems capable of automation and real-time monitoring.

## Real-World Uses of IoT

IoT is used in many industries today.

### Smart Homes

Smart bulbs, smart locks, motion sensors, and voice assistants automate home environments and improve convenience.

### Healthcare

Wearable health devices can monitor heart rate, oxygen levels, and sleep patterns in real time.

### Agriculture

Farmers use IoT sensors to monitor soil moisture, temperature, and weather conditions to improve crop production.

### Industrial Automation

Factories use IoT devices to monitor machinery, predict failures, and automate manufacturing processes.

### Smart Cities

Traffic management systems, smart parking systems, and pollution monitoring sensors help improve urban infrastructure.

### Advantages of IoT

IoT provides several advantages:

* Real-time monitoring and automation
* Remote access to devices
* Reduced human effort
* Better data collection and analytics
* Increased efficiency and productivity
* Improved decision making

## Disadvantages of IoT

Although IoT is powerful, it also introduces challenges.

* Security vulnerabilities
* Privacy concerns
* Dependence on internet connectivity
* Complex troubleshooting
* Device compatibility issues
* Increased attack surface for cyber threats

Because of these security concerns, networking and cybersecurity knowledge become very important in IoT environments.

## DIY IoT Projects You Can Build

One of the best ways to learn IoT is by building projects yourself. Some beginner-friendly IoT projects include:

* Smart temperature monitoring systems
* Motion detection alarms
* Smart plant watering systems
* Wi-Fi controlled LEDs
* Smart weather stations
* Home automation systems
* RFID access control systems
* IoT-based security cameras

In this project, we are building a temperature monitoring system that combines sensor readings, wireless communication, dashboards, and automated lighting.

## Hardware Used in This Project

For this project I am using the following components:

* ESP8266 NodeMCU Wi-Fi development board
* Keyestudio Analog Temperature Sensor
* RGB LED strip
* Jumper wires
* USB cable
* Linux machine running Node-RED and MQTT broker

The main microcontroller used in this project is an ESP8266 Wi-Fi module.

[ESP8266 NodeMCU Development Board](https://www.amazon.com.au/DIGISHUO-ESP8266-NodeMCU-Development-Micropython/dp/B097D6Y12K/?utm_source=chatgpt.com)

The ESP8266 is a low-cost Wi-Fi enabled microcontroller commonly used in IoT projects. It can connect directly to wireless networks and communicate with servers, APIs, dashboards, and other IoT devices.

## Installing the CH340 Driver

Before programming the ESP8266 board, we first need to make sure the computer can properly detect the microcontroller.

Many ESP8266 boards use the CH340 USB-to-Serial chip for communication between the computer and the microcontroller. Without the correct driver installed, the operating system may not recognize the board correctly.

You can download the CH340 driver here:

[CH340 Driver Download](https://sparks.gogo.co.nz/ch340.html?srsltid=AfmBOooENRLkk9bPYR5KB0WhJws46HBYxj_mkjgCbTa5nLk_2BgRAI5a&utm_source=chatgpt.com)

![Downloading CH340 USB-to-Serial driver required for ESP8266 communication](assets/img/blog8/0-download-ch340-driver.webp)
*Figure 1: Downloading the CH340 USB-to-Serial driver used for communication between the ESP8266 board and the computer.*

The CH340 driver acts as a bridge between the operating system and the USB serial interface on the microcontroller board. Once installed, the computer will be able to identify the ESP8266 correctly and communicate with it through a COM port.

After installing the driver, connect the ESP8266 board to the computer using a USB cable.

Open Device Manager and navigate to:

`Ports (COM & LPT)`

If everything is working correctly, you should see something similar to:

`USB-SERIAL CH340 (COM*)`

![Device Manager showing USB-SERIAL CH340 COM port](assets/img/blog8/1-device-manager-com-ports.webp)
*Figure 2: Windows Device Manager successfully detecting the ESP8266 board through the CH340 USB-to-Serial interface.*

This confirms that the computer has successfully identified the microcontroller.

## Installing Arduino IDE

Next, we need to install the Arduino IDE.

Arduino IDE is a software development environment used to write, compile, and upload code to microcontrollers such as Arduino boards and ESP8266 modules.

It provides:

* Code editor
* Compiler
* Serial monitor
* Library management
* Board management tools

You can download Arduino IDE here:

[Arduino IDE Download Guide](https://support.arduino.cc/hc/en-us/articles/360019833020-Download-and-install-Arduino-IDE?utm_source=chatgpt.com)

Once installed, we need to add support for ESP8266 boards because they are not included by default.

Navigate to:

`File -> Preferences`

Inside the **Additional Boards Manager URLs** field, paste:

```text
http://arduino.esp8266.com/stable/package_esp8266com_index.json
```

Then press **OK**.

![Adding ESP8266 board manager URL inside Arduino IDE preferences](assets/img/blog8/2-installing-esp8266-board.webp)
*Figure 3: Adding the ESP8266 package repository URL inside Arduino IDE preferences.*

Next go to:

`Tools -> Board -> Boards Manager`

![Opening the Boards Manager inside Arduino IDE](assets/img/blog8/3-accessing-boards-manager.webp)
*Figure 4: Accessing the Arduino IDE Boards Manager to install additional board packages.*

Search for:

```text
esp8266
```

Then install:

```text
esp8266 by ESP8266 Community
```

![Searching for ESP8266 package inside Boards Manager](assets/img/blog8/4-search-esp8266-community-library.webp)
*Figure 5: Searching for the ESP8266 Community board package inside Arduino IDE.*

![Installing ESP8266 Community board package](assets/img/blog8/5-installing-esp8266-community-library.webp)
*Figure 6: Installing the ESP8266 Community package required for programming ESP8266 boards.*

Once installed, select the board:

`Tools -> Board -> Generic ESP8266 Module`

Now configure the following settings:

* Upload Speed: `115200`
* Port: `COM6` (or the COM port shown in Device Manager)
* CPU Frequency: `80 MHz`

![Selecting Generic ESP8266 board](assets/img/blog8/6-selecting-generic-esp8266-board.webp)
*Figure 7: Selecting the Generic ESP8266 board profile inside Arduino IDE.*

![Selecting the correct serial COM port](assets/img/blog8/7-choosing-correct-serial-port.webp)
*Figure 8: Selecting the correct COM port used by the ESP8266 board.*

## Testing the ESP8266 with a Blink Program

Before working with sensors and wireless communication, it is always a good idea to verify that the board can successfully upload and run code.

We will start with a simple LED blinking program.

You can access all project source codes from my GitHub repository.

[Code 1]

Open Arduino IDE and paste the LED Blink code, then click the **Upload** button located in the top-left corner.

During the upload process:

* The IDE will compile the code
* The board LED may blink rapidly
* Upload progress will appear at the bottom

Once completed successfully, the output should display messages similar to:

```text
Wrote ****** bytes
Hash of data verified
Hard resetting via RTS pin
```

![Uploading LED blink code to ESP8266](assets/img/blog8/8-uploading-code-1.webp)
*Figure 9: Uploading the LED blink test program to verify the ESP8266 configuration.*

After the upload finishes, the onboard LED should turn ON and OFF every second.

This confirms that:

* The USB driver works correctly
* The board settings are correct
* The ESP8266 can successfully receive code uploads

Now we can move on to working with sensors.

## Connecting the Keyestudio Temperature Sensor

In this section, we are going to use a simple Keyestudio analog temperature sensor and display temperature readings inside the Arduino IDE Serial Monitor.

[Keyestudio Temperature Sensor](https://www.keyestudio.com/products/free-shipping-keyestudio-ds18b20-temperature-sensor-module-for-arduino?utm_source=chatgpt.com)

Before connecting any wires, disconnect the ESP8266 board from the computer.

The sensor has three pins:

* **G** → Ground
* **V** → Power
* **S** → Signal output

The ESP8266 pins used in this project are:

* **G** → Ground reference
* **3V** → 3.3V power output
* **A0** → Analog input pin used to read sensor voltage values

Use the following pin arrangement:

```text
Keyestudio Sensor Pin -> ESP8266 Pin

G -> G
V -> 3V
S -> A0
```

![Temperature sensor wiring diagram](assets/img/blog8/29-wiring-diagram-1.webp)
*Figure 10: Wiring diagram for temperature sensor*
Once the connections are complete, reconnect the board to the computer using the USB cable.

Make sure the COM port has not changed.

Next, copy the temperature reading code into Arduino IDE and upload it to the board.

After uploading:

Navigate to:

`Tools -> Serial Monitor`

Set the baud rate to:

```text
115200
```

The Serial Monitor should now display temperature readings every two seconds in both Celsius and Fahrenheit.

![Temperature readings displayed in Arduino Serial Monitor](assets/img/blog8/9-output-on-serial-monitor.webp)
*Figure 11: Temperature values received from the Keyestudio sensor and displayed inside the Arduino IDE Serial Monitor.*

## Sending Temperature Data Wirelessly Using MQTT and Node-RED

Now we are going to make the project more advanced by wirelessly transmitting sensor data to another computer and visualizing it in real time.

For this section we will use:

1. MQTT
2. Mosquitto
3. Mosquitto Clients
4. Node-RED

## What is MQTT?

MQTT (Message Queuing Telemetry Transport) is a lightweight messaging protocol commonly used in IoT systems.

It follows a publish-subscribe communication model.

Instead of devices communicating directly with each other:

* Devices publish data to a broker
* Other devices subscribe to topics
* The broker distributes the messages

MQTT is popular because it is:

* Lightweight
* Fast
* Efficient
* Ideal for low-power IoT devices

## What is Mosquitto?

Mosquitto is an open-source MQTT broker.

Its job is to:

* Receive messages
* Process subscriptions
* Forward messages to subscribers

In this project:

* ESP8266 publishes temperature data
* Mosquitto receives the data
* Node-RED subscribes to the data

## What are Mosquitto Clients?

Mosquitto Clients are command-line tools used to test MQTT communication.

Examples include:

* `mosquitto_pub`
* `mosquitto_sub`

They help test message publishing and subscriptions during troubleshooting.

## What is Node-RED?

Node-RED is a browser-based flow programming platform widely used in IoT and automation projects.

It allows users to create workflows using drag-and-drop nodes instead of writing large amounts of code.

Node-RED can:

* Receive MQTT messages
* Process data
* Control devices
* Create dashboards
* Connect APIs and databases
* Automate workflows

It is extremely popular in IoT environments because it makes rapid prototyping simple.

## Preparing the Linux Machine

Both the ESP8266 and the Linux machine must be connected to the same network.

In this setup:

* Linux machine acts as the MQTT broker
* ESP8266 sends temperature data over Wi-Fi
* Node-RED visualizes the data

Before starting, install some required packages:

```bash
sudo apt install build-essential git curl -y
```

Explanation of the packages:

* `build-essential` → Installs compiler tools required for building software
* `git` → Used for downloading repositories and version control
* `curl` → Used for downloading files and scripts from the internet
* `-y` → Automatically confirms installation prompts

![Installing build-essential, git, and curl](assets/img/blog8/10-installing-build-essential-git-curl.webp)
*Figure 12: Installing required Linux packages needed for software downloads and compilation.*

Next install Mosquitto and Mosquitto Clients:

```bash
sudo apt install mosquitto mosquitto-clients -y
```

![Installing Mosquitto MQTT broker and clients](assets/img/blog8/11-installing-mosquitto-and-client.webp)
*Figure 13: Installing the Mosquitto MQTT broker and MQTT client utilities.*

To configure Mosquitto, open the configuration file using a text editor.

I am using Mousepad:

```bash
sudo mousepad /etc/mosquitto/mosquitto.conf
```

![Opening Mosquitto configuration file](assets/img/blog8/12-configuring-mosquitto-conf-file.webp)
*Figure 14: Opening the Mosquitto configuration file for editing.*

Add the following lines to the bottom of the file:

```text
listener 1883 0.0.0.0
allow_anonymous true
```

Explanation:

* `listener 1883 0.0.0.0`

  * Tells Mosquitto to listen on port 1883
  * `0.0.0.0` means accept connections from all network interfaces

* `allow_anonymous true`

  * Allows devices to connect without authentication
  * Useful for lab environments and testing
  * Not recommended for production environments due to security risks

![Adding MQTT listener and anonymous access configuration](assets/img/blog8/13-adding-listener-and-port.webp)
*Figure 15: Configuring Mosquitto to accept MQTT connections on port 1883.*

Now enable and start the Mosquitto service:

```bash
sudo systemctl enable mosquitto.service
sudo systemctl start mosquitto.service
```

Explanation:

* `enable`

  * Starts the service automatically during boot

* `start`

  * Immediately starts the service

![Enabling and starting Mosquitto service](assets/img/blog8/14-enabling-starting-mosquitto.webp)
*Figure 16: Enabling and starting the Mosquitto MQTT broker service.*

## Installing Node-RED

Now we can install Node-RED using:

```bash
bash <(curl -sL https://github.com/node-red/linux-installers/releases/latest/download/update-nodejs-and-nodered-deb)
```

![Downloading Node-RED installer](assets/img/blog8/15-downloading-node-red.webp)
*Figure 17: Downloading the automated Node-RED installation script.*

This script downloads and installs:

* Node.js
* Node-RED
* Required dependencies

The installation may take several minutes.

![Installing Node-RED packages](assets/img/blog8/16-installing-node-red.webp)
*Figure 18: Installing Node-RED and required dependencies on Linux.*

Once installation completes, enable and start Node-RED:

```bash
sudo systemctl enable nodered.service
sudo systemctl start nodered.service
sudo systemctl status nodered.service
```

When the service status displays:

```text
active (running)
```

it confirms Node-RED is operating correctly.

![Enabling and verifying Node-RED service](assets/img/blog8/17-enabling-starting-confirming-node-red.webp)
*Figure 19: Enabling, starting, and confirming the Node-RED service status.*

Before accessing Node-RED, allow MQTT traffic through the firewall:

```bash
sudo ufw allow 1883
```

You can then access Node-RED in the browser:

```text
http://127.0.0.1:1880
```

![Accessing Node-RED dashboard in browser](assets/img/blog8/18-accessing-node-red-dashboard.webp)
*Figure 20: Accessing the Node-RED editor dashboard through a web browser.*

## Creating the Node-RED Workflow

Node-RED workflows are created using nodes connected together.

In this project, the workflow will:

1. Receive MQTT data
2. Convert string data into numeric format
3. Display readings in a real-time gauge

## MQTT In Node

First drag an **mqtt in** node into the workspace.

The MQTT In node subscribes to MQTT topics and receives messages from the broker.

Configure:

* Server: `127.0.0.1:1883`
* Action: `Subscribe to single topic`
* Topic: `home/sensor/tempC`

![Configuring MQTT In node in Node-RED](assets/img/blog8/19-mqtt-in-node.webp)
*Figure 21: Configuring the MQTT In node to subscribe to the temperature topic.*

## Function Node

The ESP8266 sends data as strings.

The Gauge node requires numeric values.

To convert strings into numbers, add a **Function** node.

The Function node allows custom JavaScript processing inside Node-RED flows.

In this project, it converts string values into floating-point numbers.

![Function node converting string to float](assets/img/blog8/20-function-node.webp)
*Figure 22: Using a Function node to convert MQTT string data into numeric values.*

## Gauge Node

Next add a **Gauge** node.

The Gauge node visually displays sensor readings in real time.

Configure:

* Label: `Temperature`
* Value format: `{{value}}°C`
* Range: `0 - 100`

![Default gauge node configuration](assets/img/blog8/21-default-gauge-node.webp)
*Figure 23: Default Gauge node before customization.*

![Customized temperature gauge settings](assets/img/blog8/22-refined-gauge-node.webp)
*Figure 24: Customized Gauge node configured for temperature monitoring.*

The completed workflow should look similar to this:

![Final Node-RED workflow](assets/img/blog8/23-final-flow.webp)
*Figure 25: Final Node-RED workflow used to receive and display MQTT temperature data.*

## Installing MQTT Library for Arduino IDE

Before uploading the ESP8266 MQTT code, install the required MQTT library.

Navigate to:

`Tools -> Library Manager`

Search for:

```text
PubSubClient
```

Install:

```text
PubSubClient by Nick O'Leary
```

The PubSubClient library allows Arduino and ESP8266 devices to communicate with MQTT brokers.

![Installing PubSubClient library](assets/img/blog8/24-installing-pub-sub.webp)
*Figure 26: Installing the PubSubClient MQTT library inside Arduino IDE.*

Now copy the Node-RED temperature code into Arduino IDE and upload it to the ESP8266.

After uploading, open Serial Monitor to confirm temperature data is being published successfully.

To access the dashboard UI, open:

```text
http://127.0.0.1/ui
```

This page displays the Node-RED Dashboard interface where the Gauge widget is rendered.

![Temperature readings shown in serial monitor while publishing MQTT data](assets/img/blog8/25-output-in-serial-monitor.webp)
*Figure 27: ESP8266 publishing temperature readings while displaying them in Serial Monitor.*

![Temperature gauge inside Node-RED dashboard](assets/img/blog8/26-output-in-node-red-ui.webp)
*Figure 28: Real-time temperature visualization inside the Node-RED dashboard.*

![Additional Node-RED dashboard output](assets/img/blog8/27-output-in-node-red-ui.webp)
*Figure 29: Live MQTT temperature data displayed through the Node-RED user interface.*

## RGB LED Temperature Indicator

For the final section of this project, we will connect an RGB LED strip and configure it to change colors according to temperature readings.

The LED strip has three wires:

* Red → Power
* White → Ground
* Green → Data signal

Use the following pin arrangement:

```text
D4 -> Green wire
USB Black -> Ground
USB Red -> Power
```

![LED strip wiring diagram](assets/img/blog8/30-wiring-diagram-2.webp)
*Figure 30: Wiring diagram for the LED strip and the temperature sensor*

The LED strip requires external 5V power.

One easy method is using a USB cable:

* USB red wire → 5V
* USB black wire → Ground

The ESP8266 data pin controls the LED colors while the external USB power provides sufficient current for the LEDs.

Before uploading the LED control code, install the required library:

```text
Adafruit NeoPixel by Adafruit
```

This library allows microcontrollers to control addressable RGB LEDs such as NeoPixels and WS2812 LED strips.

![Installing Adafruit NeoPixel library](assets/img/blog8/28-installing-neo-pixel-driver.webp)
*Figure 31: Installing the Adafruit NeoPixel library required for RGB LED control.*

Once installed, upload the LED temperature code.

## Demonstrating the Temperature-Based RGB LED System

Now that the project is fully configured, it's time to see everything working together.

After uploading the final code, the ESP8266 continuously reads temperature data from the Keyestudio temperature sensor and performs three tasks simultaneously:

* Displays the temperature readings in the Arduino IDE Serial Monitor
* Publishes the readings to the MQTT broker
* Updates the Node-RED dashboard gauge in real time
* Changes the RGB LED color based on the detected temperature

The LED color thresholds configured in this project are:

| Temperature Range | LED Color |
| ----------------- | --------- |
| Below 10°C        | Green     |
| 11°C – 15°C       | Yellow    |
| Above 16°C        | Red       |

As the surrounding temperature changes, the LED automatically updates its color without requiring any user interaction. This demonstrates a simple form of automation, which is one of the core concepts behind IoT systems.

The Node-RED dashboard also updates in real time, allowing temperature values to be monitored remotely from another computer connected to the same network.

### Video Demonstration

The following video demonstrates the completed project in action.

<video width="100%" autoplay loop muted playsinline controls preload="metadata">
  <source src="{{ '/assets/img/blog8/optimized_output.mp4' | relative_url }}" type="video/mp4">
  Your browser does not support the video tag.
</video>

*Video 1: Demonstration of the complete IoT temperature monitoring system showing real-time temperature readings, MQTT communication, Node-RED visualization, and RGB LED color changes based on temperature thresholds.*

This simple demonstration highlights how IoT devices can collect data from sensors, communicate wirelessly over a network, visualize information on dashboards, and trigger automated actions based on predefined conditions.

## Conclusion

In this project, we successfully built a complete IoT temperature monitoring system using an ESP8266 Wi-Fi module, a Keyestudio temperature sensor, MQTT, Mosquitto, Node-RED, and an RGB LED strip.

We started by configuring the ESP8266 development environment and verifying communication with the board using a simple LED blink program. We then learned how to read analog temperature data and display it through the Arduino IDE Serial Monitor.

Next, we expanded the project by introducing MQTT communication and Node-RED dashboards. By sending temperature readings wirelessly across the network, we transformed a simple sensor project into a true IoT solution capable of remote monitoring and visualization.

Finally, we added an RGB LED strip that responds dynamically to temperature changes. This demonstrated how IoT systems can not only collect and display data but also make automated decisions based on real-world conditions.

Throughout this project, we explored several important technologies commonly used in professional IoT environments:

* ESP8266 microcontrollers
* Analog sensors
* Serial communication
* MQTT messaging
* Mosquitto brokers
* Node-RED dashboards
* Real-time monitoring
* Automation and control systems

Although this project is relatively simple, the same concepts are used in smart homes, industrial monitoring systems, environmental monitoring platforms, agricultural automation solutions, and many other real-world IoT deployments.

As a next step, you could expand this project by:

* Logging temperature readings to a database
* Creating historical graphs and reports
* Sending email or mobile alerts
* Integrating cloud platforms such as AWS IoT or Azure IoT
* Adding humidity, motion, or light sensors
* Securing MQTT communications using authentication and TLS encryption

IoT is an exciting field that combines electronics, networking, programming, cybersecurity, and automation into a single ecosystem. The best way to learn is by building projects, experimenting with new ideas, and gradually increasing complexity.

I hope this project has provided a practical introduction to IoT and inspired you to start creating your own connected devices and smart automation solutions.

Thank you for reading, and happy building!
