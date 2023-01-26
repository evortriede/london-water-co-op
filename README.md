# london-water-co-op
Hardware and software for automation of the London Water Cooperative WTP

## Introduction
The London Water Cooperative (LWC) is a small public drinking water system in London Springs, Oregon. LWC has 22 metered connections serving aproximately 60 persons, a church and a grange. LWC has two water storage tanks totaling ~23,500 gallons. The Water Treatment Plant (WTP) draws water from a creek and is run in batches as needed (typically every four to six days) treating about 18,000 gallons per batch. 

The WTP is very old and is slated for a complete rebuild; however, that rebuild will take some time to complete and in the meantime there are a number of issues that have been screaming for more timely addressing. These include:

- There was no way to tell if there was a major leak apart from turning on a spigot and observing that there was no water. A system for remotely monitoring instantanious water usage and making audible and visual alerts when water usage is abnormally high was needed.
- There was no way to tell how much water was in the storage tanks apart from physically going to the WTP. A system for remotely monitoring the water level in the storage tanks was needed.
- The Oregon Health Authority (OHA) requires monthly reports detailing the turbidity of treated water. Metrics for the report were collected automatically but had to be manually copied to the reports, a process that was very time consuming and error prone. Also, the granularity of the collected metrics was inadequate. A system for collecting fine graned metrics for turbidity and volume of water being treated was needed. Also, an automated system for creating the OHA reports from the collected metrics was needed.
- Part of the treatment process is adding chlorine to the treated water. The process of determining how much chlorine to add to the water being treated from a stoichiometric point of view is a complex subject, but generally speaking, that is the starting point for setting up the concentration of chlorine that is pumped into the treatment stream and at what rate it is pumped. The problem is that no matter how perfectly it gets dialed in at the start of a batch, it is impossible to uniformly regulate the volume of water being processed for an entire run. If the volume goes down from where things were perfect, the end result is over chlorination. If the volume increases, the result is under chlorination. An automated solution for measuring the chlorine concentration in the treated water throughout an entire run and adjusting the pump accordingly was needed.

There are a number of other issues that further complicate the core requirements:

- There is no Internet or cellphone service at the WTP
- By and large the LWC membership is at or below the poverty line with a few exceptions (funding for rebuilding the plant is from an EPA grant)
- The programs that are run on the PLCs that run the plant are virtualy black boxes. If you want to know what they are doing you have to reverse engineer everything.
- LWC is entirely a volunteer operation. The only reason that it works is that we really like being able to flush our toilets and other stuff that running water lets us do.

These issues have been addressed by creatively employing a handful of inexpensive microcontrollers that interface with existing PLCs, measure current for pressure pumps, communicate over short distances with WiFi and over long distances with LoRa. Also, a donated Hach CL17 automatic chlorine meter and a controllable (via 0-5v) chemical pump have made it all possible. The various repositories in this account contain the hardware designs and software which implement the solutions.

## System Map
