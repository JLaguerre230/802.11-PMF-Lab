# Protected Management Frames

## Demystifying Protected Management Frames (PMF): A Hands-On 802.11 Analysis

Protected Management Frames (PMF), originally introduced by IEEE 802.11w and later incorporated into the IEEE 802.11 standard, provide protection against attacks that exploit certain unprotected management frames.

PMF protects a set of robust management frames and extends the security protections already provided to data frames through IEEE 802.11i.

PMF is required when using WPA3 and Enhanced Open (OWE).

PMF can be configured as Disabled, Capable/Optional, or Required, depending on the wireless implementation. These different modes provide compatibility with client devices that may or may not support PMF.

For example, an older client device may not support PMF, while a newer device may support it. If PMF is configured as Capable/Optional, both devices may still be able to join the wireless network. A PMF-capable client can negotiate and use PMF, while a client that does not support PMF can connect without it.


## Lab Demonstration

I decided to demonstrate, using a minimal lab setup, how access points with different PMF configurations appear in Wireshark packet captures. I will also demonstrate how to configure an access point with basic PMF settings for lab purposes.

### Requirements

* Linux
* One wireless network adapter capable of monitor mode
* Wireshark

The first step is to make sure our wireless interface is operating in monitor mode. Monitor mode allows the interface to capture IEEE 802.11 frames transmitted on the selected channel, including frames that are not specifically addressed to our device. 


<img width="835" height="217" alt="1_iw_dev_wlan_STEP_1" src="https://github.com/user-attachments/assets/a0b9ac6f-a8f1-4ec0-8d2b-140c1b4e2017" />


We will use WLAN1 for our access point, while WLAN99 will operate in monitor mode to capture the beacon management frames transmitted by the WLAN1 access point. 

<img width="722" height="173" alt="2_wlan99_create_STEP_2" src="https://github.com/user-attachments/assets/4b09b6b9-b81e-46bc-b382-27ae4739ba50" />


We then need to create a hostapd configuration file, where the necessary information for creating our access point will be stored.

For this first hostapd configuration, we created a network using Wired Equivalent Privacy (WEP). WEP is an obsolete and insecure wireless security mechanism that predates Protected Management Frames and does not support PMF. It also contains numerous security weaknesses, including vulnerabilities related to its initialization vector (IV) and RC4 implementation. 


<img width="324" height="49" alt="3_create_hostapd_STEP_3" src="https://github.com/user-attachments/assets/bc35088b-4af0-42fb-ba60-b29425081aa3" />
<img width="243" height="309" alt="4_hostapd_wep_STEP_4" src="https://github.com/user-attachments/assets/b9fcd7b2-cb47-4181-8753-1eff055c0dc3" />



We then need to launch hostapd using our configuration file. 

<img width="414" height="104" alt="5_lauching_hostapd_STEP_5" src="https://github.com/user-attachments/assets/c8640d77-2afb-4fd4-b62d-b25bc6e6cf46" />


There are multiple ways to gather and analyze information related to Protected Management Frames. We could use tcpdump, TShark, or even a Python script using Scapy. For this demonstration, however, we decided to use the well-known tool airodump-ng. In this command, `-c` specifies the channel we want to monitor, while `-w` specifies the prefix of the files where the captured packets will be stored. 


<img width="428" height="46" alt="6_lauching_the_airodump-ng_STEP_6" src="https://github.com/user-attachments/assets/b950f599-c1c4-4903-8dac-4517c4b46558" />



As we can see in the output, our test access point has already transmitted 177 beacon frames. We can now stop the capture and begin analyzing it using another well-known tool, Wireshark.

Airodump-ng creates several files containing different types of information about the capture. Some of these formats can also be used by tools such as Kismet, a wireless monitoring platform that can collect and display wireless network information through a web interface. In our case, the file we are interested in is `wep-01.cap`, which we will open with Wireshark. 

<img width="757" height="200" alt="7_airodump_capture_STEP_7" src="https://github.com/user-attachments/assets/479d45ce-6d94-4b1c-b324-74ccdcf8e590" />
<img width="799" height="125" alt="8_entering_wireshark_STEP_8" src="https://github.com/user-attachments/assets/b582f039-8bb8-470d-a088-239835a2b02f" />



Now that we are inside Wireshark, we can apply a filter that allows us to display only beacon frames. We can then examine the different fields available within the frames.

In our WEP capture, the Robust Security Network (RSN) Information Element is absent. This is expected because WEP predates RSN. Instead, we can examine the Capability Information field and look at the Privacy bit. When the Privacy bit is set to `1`, the BSS indicates that data confidentiality is required. When it is set to `0`, the BSS does not advertise that privacy capability. The Privacy bit alone does not identify the specific security protocol being used. 

<img width="1507" height="162" alt="09_how_to _show_beacon" src="https://github.com/user-attachments/assets/8a1c7706-f37c-4e64-bd20-a2e17ef32db5" />
<img width="802" height="249" alt="9_showing_the_wep_capture_STEP_9" src="https://github.com/user-attachments/assets/9b5a12cd-b794-4062-a3a4-c4f7d546615b" />


For the next test, we created a WPA2-Personal network using CCMP. We followed the same steps to create the access point and capture its beacon frames.

As we can observe in Wireshark, both the Management Frame Protection Capable (MFPC) and Management Frame Protection Required (MFPR) bits are set to `0`, since PMF was not enabled in this configuration.

This means:

**MFPC = 0 — Not capable**
**MFPR = 0 — Not required**

The PTKSA Replay Counter field also advertises support for 16 replay counters. This value represents the number of replay counters supported by the device; it is not the current packet sequence number or replay-counter value. 

<img width="284" height="335" alt="10_hostapd_wpa2_STEP_10" src="https://github.com/user-attachments/assets/b3836c23-823c-455d-a5a8-4701281080e2" />
<img width="1313" height="757" alt="11_wpa2_IE_STEP_11" src="https://github.com/user-attachments/assets/2b69f53d-3b94-4cf7-a3e2-42b2ba58fa25" />



For the next configuration, we created an access point with Protected Management Frames set to Capable/Optional. This means that the access point supports PMF but does not require every client to use it.

We used the same technique to capture and analyze the beacon frame. In Wireshark, we can now see that the Management Frame Protection Capable bit is set to `1`, while the Management Frame Protection Required bit remains set to `0`.

This means:

**MFPC = 1 — Capable**
**MFPR = 0 — Not required**

A PMF-capable client can negotiate and use PMF, while a client that does not support PMF may still be able to connect. 

<img width="354" height="391" alt="12_hostapd_wpa3_1_STEP_12" src="https://github.com/user-attachments/assets/0b2b60a8-32bd-466d-b509-1f8c0e33dd28" />
<img width="1309" height="734" alt="13_wpa3_1_IE_STEP_13" src="https://github.com/user-attachments/assets/a6462da6-af22-4dd8-959a-3a246f448b21" />


Finally, we created an access point with Protected Management Frames set to Required. We repeated the same capture and analysis process.

In Wireshark, we can now see that both the Management Frame Protection Capable and Management Frame Protection Required bits are set to `1`.

This means:

**MFPC = 1 — Capable**
**MFPR = 1 — Required**

Only clients that support PMF can successfully connect to a BSS where PMF is required.

Implementing PMF as Required in an environment containing legacy devices that do not support it can therefore prevent those devices from connecting. In environments containing both PMF-capable and legacy devices, administrators may instead use PMF in Capable/Optional mode when compatibility is necessary.

PMF protects certain robust management frames, including deauthentication and disassociation frames, against forgery and replay after PMF has been negotiated. However, PMF does not prevent all forms of wireless interference or denial-of-service attacks, such as RF jamming. 

<img width="331" height="349" alt="14_hostapd_wpa3 2_STEP_14" src="https://github.com/user-attachments/assets/a5f8fb3b-4bbd-41db-99f1-9442c39a9b69" />
<img width="1273" height="732" alt="15_wireshark_wpa2_2_STEP_15" src="https://github.com/user-attachments/assets/c8a41caf-a024-4f78-8435-a2b47cc2cf00" />
<img width="838" height="210" alt="Capture d’écran 2026-09-15 011805" src="https://github.com/user-attachments/assets/48274295-31c8-4441-a090-957eb8b66b0e" />
<img width="1160" height="251" alt="Capture d’écran 2026-09-15 011817" src="https://github.com/user-attachments/assets/716f1f79-d3f1-47fe-bda2-8584eed12367" />
