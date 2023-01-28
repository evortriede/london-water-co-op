# london-water-co-op
Hardware and software for automation of the London Water Cooperative WTP

![LWC WTP](/assets/LWC-WTP.png)

## Introduction
The London Water Cooperative (LWC) is a small public drinking water system in London Springs, Oregon. LWC has 22 metered connections serving approximately 60 persons, a church and a grange. LWC has two water storage tanks totaling ~23,500 gallons. The Water Treatment Plant (WTP) draws water from a creek and is run in batches as needed (typically every four to six days) treating about 18,000 gallons per batch. 

The WTP is very old and is slated for a complete rebuild; however, that rebuild will take some time to complete and in the meantime, there are a number of issues that have been screaming for more timely addressing. These include:

- There was no way to tell if there was a major leak apart from turning on a spigot and observing that there was no water. A system for remotely monitoring instantaneous water usage and making audible and visual alerts when water usage is abnormally high was needed.
- There was no way to tell how much water was in the storage tanks apart from physically going to the WTP. A system for remotely monitoring the water level in the storage tanks was needed.
- The Oregon Health Authority (OHA) requires monthly reports detailing the turbidity of treated water. Metrics for the report were collected automatically but had to be manually copied to the reports, a process that was very time consuming and error prone. Also, the granularity of the collected metrics was inadequate. A system for collecting fine grained metrics for turbidity and volume of water being treated was needed. Also, an automated system for creating the OHA reports from the collected metrics was needed.
- Part of the treatment process is adding chlorine to the treated water. The process of determining how much chlorine to add to the water being treated from a stoichiometric point of view is a complex subject, but generally speaking, that is the starting point for setting up the concentration of chlorine that is pumped into the treatment stream and at what rate it is pumped. The problem is that no matter how perfectly it gets dialed in at the start of a batch, it is impossible to uniformly regulate the volume of water being processed for an entire run. If the volume goes down from where things were perfect, the end result is over chlorination. If the volume increases, the result is under chlorination. An automated solution for measuring the chlorine concentration in the treated water throughout an entire run and adjusting the pump accordingly was needed.

There are a number of other issues that further complicate the core requirements:

- There is no Internet or cellphone service at the WTP
- By and large the LWC membership is at or below the poverty line with a few exceptions (funding for rebuilding the plant is from an EPA grant)
- The programs that are run on the PLCs that run the plant are virtually black boxes. If you want to know what they are doing you have to reverse engineer everything.
- LWC is entirely a volunteer operation. The only reason that it works is that we really like being able to flush our toilets and other stuff that running water lets us do.

These issues have been addressed by creatively employing a handful of inexpensive microcontrollers that interface with existing PLCs, measure current for pressure pumps, communicate over short distances with WiFi and over long distances with LoRa. Also, a donated Hach CL17 automatic chlorine meter and a controllable (via 0-5v) chemical pump have made it all possible. The various repositories in this account contain the hardware designs and software which implement the solutions.

## System Map

![LWC System Map](/assets/LWC-System-Map.png)

### At the WTP

#### PLC

The WTP is controlled by a Modicon 984 Programable Logic Controller (`PLC`) [Modicon PLC](/assets/984_systemmanual.pdf)
. Its main function in the WTP is to open and close automated valves that control the flow of water through the plant. For all practical purposes, it has three modes:

- Off: No water treatment happening (There shouldn't be any water coming into the plant, but if there is it will be routed to the drain.)
- On: Water is being treated (The automated valves are set such that water coming into the plant is sent through the filters and the chemical pumps are energized.)
- Backflush: The automated valves are set such that finished water flows backward through the filters and out the drain.

The reason for mentioning the `PLC` here is that the water storage `Tank Level Transducer` and the `Turbidity Meter` are connected to it and can be read by the `Current Monitor`.

#### Current Monitor

The `Current Monitor` and remotely located `Current Recorder` were the first components to be implemented and both utilize the Heltec ESP32 LoRa V2 microcontroller board. As the names imply, the mission was to monitor and record the off-to-on and on-to-off transitions of the pressure pump that supplies water to the consumers in order to detect high water usage that could be indicative of a leak in the system. The `Current Monitor` communicates with the `Current Recorder` using LoRa which is a long-range radio technology (Google it). Today, in addition to monitoring the pump transitions, the `Current Monitor` uses Modbus/TCP to read `PLC` holding registers to obtain tank level and turbidity readings. It also uses TCP/IP to communicate with the `Pump Controller` to get and set pump settings and to get chlorine readings.

#### Pump Controller

The `Pump Controller`, as its name implies, controls the chlorine pump. It does so by monitoring the recorder output of a Hach CL17 chlorine meter via 4-20mA current loop and outputting proportional voltage to the pump's 0-5 Volt control input. It is implemented using a generic ESP32 Microcontroller Development Board. It has automatic and manual modes of operation. In automatic mode, it automatically adjusts the pump speed based on the `CL17` readings and in manual mode it adjusts the pump to manual input. It connects to the `Current Monitor` through WiFi via a Telnet connection over which it sends `CL17` readings and pump settings and receives pump settings originating from the `Current Recorder` HTTP interface. It implements an HTTP interface for monitoring, configuration and manual input.

#### CL17

This is a Hach CL17 Chlorine meter [Hach CL17](/assets/CL17_Manual_Edition_6.pdf). It is connected to the `Pump Controller` via 4-20mA current loop.

#### Turbidimeter

This is a Hach 1720D/L Low Range Process Turbidimeter [Turbidimeter](/assets/Turbidimeter_Manual.pdf). It is connected to the `PLC` via 4-20mA current loop.

#### Chlorine Pump

This is a Kamoer DIPump550 peristaltic pump [DIPump550](/assets/Chlorine_Pump_manual.pdf). The 0-5 Volt input on the `Pump` is connected to a analog output pin of the `Pump Controller`.

### Off Site (from the WTP)

The WTP has no cell phone or Internet access, so any data transfer between the WTP and anywhere else is performed using LoRa. Bidirectional data transfer happens between the `Current Monitor` at the WTP and the `Current Recorder` which is Off Site. Other components, primarilly the `LWC Monitor`, listen in on the LoRa conversation and can inject the data into the Internet where it can be accessed remotely.

#### Current Recorder

The primary purpose of the `Current Recorder` is to receive LoRa messages from the `Current Monitor` and send them to a `PC or Raspberry Pi` that runs a Java program to record the data permenantly. It also has a HTTP interface with pages for configuration and for sending `Pump` settings to the `Current Monitor` (via LoRa) which forwards them to the `Pump Controller` (via Telnet).

#### LWC Monitor

The `LWC Monitor` listens in on the LoRa conversation between the `Current Monitor` and `Current Recorder`. It displays summary information (Tank level, Gallons Per Hour and Gallons per day) on the OLED display of its Heltec ESP32 LoRa V2 microcontroller. It sports an RGB LED and a buzzer for other status indications (high water usage, loss of communication). It can function anywhere that the LoRa signal is available (i.e., within a few kilometers of the WTP and `Current Recorder`).  The `LWC Monitor` can join a local WiFi AP and/or act as an AP itself. It has a HTTP interface for configuration and information display. Given a WiFi connection with Internet access, the `LWC Monitor` can send the WTP metrics to a website where it can be accessed from virtually anywhere.

#### Remote Web Server

The current Off Site, setting where the `Current Recorder` lives, is sufficiently firewalled such that it cannot be accessed from the Internet. Since it is desired that status information from the WTP be available from the Internet, an Internet accessible server is required. Virtually any server that can execute a PHP script will suffice and all that is required is the installation of two scripts, one for writing the data and one for retrieving the data. A properly configured instance of the `LWC Monitor` with Internet access will send an HTTP request to execute the script to store the data whenever the data changes and `lwcmon.html` will retrieve the data using the other script.

#### lwcmon.html

This is an HTML document that, when loaded in a browser, displays the status information that `LWC Monitor` stores on the `Remote Web Server`. It is completely transportable and can be used from anywhere that there is Internet access.

#### PC or Raspberry Pi

The `Current Recorder` is probably misnamed in that it doesn't record anything. Instead, it waits for something to make a Telnet connection and then forwards all of the LoRa data over that connection. The 'something' that makes that connection is a Java program running on a computer. That computer is currently a Raspberry Pi but as of 2/1/2023 will be a Windows PC. The Java program stores the LoRa data in files on disk (or SSD). There are other Java programs for processing the data to produce the report that OHA requires.

## Repositories

In Alphabetical order. See the README.md files in each repository for details.

### AsyncLWCMonitor

The `LWC Monitor` firmware and remote server scripts.

### CurrentMonitor

The `Current Monitor` firmware.

### CurrentRecorder

The `Current Recorder` firmware.

### FSM

An Arduino library for a Finite State Machine that is required for the `GenericProtocol` library.

### GenericProtocol

An Arduino library for implementing a very simple protocol to ensure reliable delivery of messages between two endpoints. This protocol is used for all of the LoRa communications.

### LWC_Java

The Java programs for storing the metrics and creating the OHA reports.

### PumpController

The `Pump Controller` firmware.

