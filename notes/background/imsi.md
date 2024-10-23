# International Mobile Subscriber Number - IMSI

## Motivation

### Cell Selection & Timing Advance

- Cellular network architecture requires to have cells (hexagonal tiling)
- TDMA is used for efficient sharing for air time, while FDMA is for assigning a specific frequency
    - Together: assignment of resource blocks and resource elements on a resource grid
- However, it is necessary to manage the UE's to not overlap in time 
    - One UE can be far away while another is close
    - Time of arrival for the radio wave can lead to interference and would therefore nullify the connection

## Functionality of an IMSI Catcher

Example focuses on GSM, but the same/similar concepts apply for CDMA/UMTS/LTE:

1. Find the strongest cell, and parse its neighbor list
2. Find the "weakest" neighbor - with the weakest SNR
3. Spoof the weakest neighbor with a different Location Area Code (LAC)
4. UEs will try to connect and send a **Location Area Update**
    - They switched from one Location Area to another and want to advertise this to the Core Network
    - Otherwise the network cannot page the phone correctly (e.g. on a phone call)
    - In this process the UE will first **send the TMSI (temporary mobile subscriber number) to not send the IMSI/IMEI**
5. Then the rouge base station will just reject the TMSI and ask for IMSI/IMEI, where the UE will provide them without prior authentication


### Why is this possible?

- Previous generations of cellular did not account for rouge basestations
- No mutual authentication; only phones authencate to basestations
- To prohibit this: disable 2G 
