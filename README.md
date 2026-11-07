## IoT-Device-Discovery-Network-Traffic-Analysis
## Project type: Blue Team / SOC Analyst
## Focus: Passive PCAP analysis, IoT protocol identification, network visibility
## Tools: tcpdump, grep, wc
## Assessment environment: Praxexa IoT Device Discovery exercise

## Overview
This project documents a defensive investigation of a supplied smart-building network packet-capture transcript. The goal is to get oriented in an unfamiliar capture, identify traffic associated with common IoT protocols, distinguish devices that share a protocol, and determine which protocol exposes device names in the packet output.

## The investigation is passive:
tcpdump -r reads an existing capture file. The commands in this repository do not scan, exploit, or send traffic to network devices.
Evidence note: The original PCAP was not included with the material used to prepare this repository. Packet counts and observations below are transcribed from the assessment output supplied by the learner. They have not been independently reproduced against the original capture.

## Objectives
- Inspect the first ten packets before applying filters.
- Identify likely MQTT traffic on TCP port 1883.
- Isolate the thermostat's MQTT traffic by host.
- Identify MQTT carried over TLS on TCP port 8883.
- Find door-controller name announcements in mDNS output.
- Count CoAP traffic on UDP port 5683.
- Explain why mDNS exposed device names in clear text in this exercise.

## Repository layout
iot-device-discovery-soc/
├── README.md
├── .gitignore
├── LICENSE
├── evidence/
│   └── assessment-evidence.md
├── reports/
│   └── soc-investigation-report.md
└── screenshots/
    └── README.md

## Environment and assumptions
The assessment uses this capture path:
```
/var/captures/iot-lan-sweep.pcap
```
The commands below assume tcpdump, grep, and wc are installed and that the analyst has permission to read the capture. Replace the path if your local copy is stored elsewhere.
Investigation workflow
## 1. Establish a baseline before filtering
```
tcpdump -n -c 10 -r /var/captures/iot-lan-sweep.pcap
```
## Assessment result: 498 packets.

TCP port 1883 is the conventional default for unencrypted MQTT. This is a useful indicator, but a port number alone does not prove the application protocol. Confirm with packet contents or protocol dissection when the PCAP is available.
## 2. Count likely cleartext MQTT traffic
```
tcpdump -n -r /var/captures/iot-lan-sweep.pcap 'tcp and port 1883' | wc -l
```
## Assessment result: 498 packets.

TCP port 1883 is the conventional default for unencrypted MQTT. This is a useful indicator, but a port number alone does not prove the application protocol. Confirm with packet contents or protocol dissection when the PCAP is available.
## 3. Isolate the thermostat
```
tcpdump -n -r /var/captures/iot-lan-sweep.pcap 'host 10.20.5.21 and port 1883' | wc -l
```
## Assessment result: 336 packets.

The host condition narrows the result to traffic involving 10.20.5.21 on port 1883. The exercise identifies this address as the thermostat.
## 4. Identify MQTT over TLS
```
tcpdump -n -r /var/captures/iot-lan-sweep.pcap 'tcp and port 8883' | wc -l
```
## Assessment result: 288 packets.

TCP port 8883 is commonly used for MQTT over TLS. TLS protects application payloads in transit, so a packet listing generally reveals metadata such as endpoints, ports, timing, and packet sizes, but not the protected MQTT messages.
## 5. Find the door-controller mDNS announcements
```
tcpdump -n -r /var/captures/iot-lan-sweep.pcap 'udp and port 5353' | grep -c 'door-ctrl-01'
```
## Assessment result: 17 matching lines.

mDNS commonly uses UDP port 5353 for local-link name and service discovery. The output in this exercise includes the device name door-ctrl-01, making mDNS the protocol that visibly identifies the device by name in the supplied transcript.
## 6. Count CoAP traffic
```
tcpdump -n -r /var/captures/iot-lan-sweep.pcap 'udp and port 5683' | wc -l
```
## Assessment result: 89 packets.

UDP port 5683 is the conventional default for CoAP. As with MQTT, confirm the protocol from packet contents when possible rather than treating the port as definitive proof.
Findings at a glance
| Traffic / observation | Filter or evidence | Assessment result |
|---|---|---:|
| First packets | `tcpdump -n -c 10 -r ...` | 10 packets inspected |
| MQTT candidate traffic | TCP port 1883 | 498 packets |
| Thermostat MQTT candidate traffic | Host `10.20.5.21` and port 1883 | 336 packets |
| MQTT over TLS candidate traffic | TCP port 8883 | 288 packets |
| Door-controller mDNS name matches | UDP port 5353 + `door-ctrl-01` | 17 matching lines |
| CoAP candidate traffic | UDP port 5683 | 89 packets |
| Protocol exposing device name in supplied output | mDNS | Identified |

## Conclusion
The assessment output indicates traffic consistent with MQTT on TCP/1883, MQTT over TLS on TCP/8883, mDNS on UDP/5353, and CoAP on UDP/5683. The thermostat at 10.20.5.21 accounts for 336 packets matching the host-and-port filter. The supplied mDNS output contains 17 lines matching door-ctrl-01.

Answer to the final assessment question: mDNS. It is the protocol that carried a device name in clear text in the provided tcpdump output.
Important analysis limitations
- The original PCAP was not supplied for independent validation.
- Port-based filters identify traffic using conventional ports; they do not conclusively identify an application protocol.
- A grep -c result counts matching output lines, not necessarily unique devices or unique DNS records.
- Packet counts are not device counts.
- The first-ten-packet sample is only a small view of the capture and may not represent all traffic.
- TLS encrypts application data, but network metadata remains observable.
  
## Defensive recommendations
1. Maintain an authorized inventory of IoT devices, IP addresses, owners, and expected services.
2. Segment IoT devices from user workstations and sensitive server networks.
3. Restrict MQTT broker access to approved devices and networks.
4. Prefer MQTT over TLS where supported; manage certificates and credentials securely.
5. Review mDNS exposure and limit discovery traffic to networks where it is needed.
6. Monitor unexpected use of MQTT, CoAP, and discovery protocols.
7. Preserve original PCAPs, timestamps, hashes, and analysis notes for repeatable investigations.
   
## Reproduce the analysis
1. Obtain the authorized assessment PCAP.
2. Verify the file path and, where required, record its SHA-256 hash.
3. Run the commands in the workflow above.
4. Compare your results with evidence/assessment-evidence.md.
5. Record any differences rather than changing results to match the transcript.

## Disclaimer
This repository is for educational and defensive network-analysis purposes. Only inspect packet captures you are authorized to access. The documented workflow reads an existing capture and does not perform exploitation or active network scanning.
