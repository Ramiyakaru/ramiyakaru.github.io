---
title: "Installing and Configuring Suricata IDS on pfSense"
date: 2026-06-12 +1030
categories: [Cybersecurity, Networking, Homelab]
tags: [IDS, IPS, pfSense, Suricata, Proxmox, Security-Monitoring]
---

## Introduction

Modern networks face constant threats ranging from simple port scans to complex multi-stage exploits. To defend critical infrastructure against these evolving tactics, organizations rely heavily on Intrusion Detection and Prevention Systems (IDS/IPS). These security engines passively analyze network traffic in real time, looking for signs of a breach or vulnerability exploitation.

In this project, we will deploy a fully functional Intrusion Detection System using **Suricata** hosted directly on a **pfSense** firewall within a **Proxmox VE** virtual environment. Once configured, we will put our defensive architecture to the test by launching real-world simulated attacks from an external attack machine.

To evaluate Suricata's detection capabilities across different layers, we will direct our attacks toward several industry-standard vulnerable targets hosted in our lab, including:

- **DVWA (Damn Vulnerable Web Application)**
- **OWASP Juice Shop**
- **Metasploitable 2 and 3**

you can learn how to install these VMs on Proxmox on my other blog posts.

By the end of this guide, you will see exactly how Suricata parses raw network packets to detect and log malicious activity in real time.

## Understanding IDS and IPS Mechanics

Before diving into the implementation, it is vital to understand the structural and operational differences between network monitoring tools.

### Intrusion Detection System (IDS)

An **Intrusion Detection System (IDS)** is a passive security tool that monitors network traffic or system activity for malicious behavior, policy violations, or known threat signatures. An IDS operates out-of-band or on a copy of the traffic. It analyzes data packets, logs anomalies, and fires off real-time alerts to security administrators, but it does **not** alter or drop traffic to stop an attack.

Think of an IDS like a **security camera system**: it provides complete visibility and records everything that occurs, but it cannot physically step in to stop an intruder.

### Intrusion Prevention System (IPS)

An **Intrusion Prevention System (IPS)** takes defensive security an active step further. Positioned in-line with network traffic, an IPS scans packets as they pass through the device. If a packet matches a known malicious signature or violates a security rule, the IPS immediately blocks or drops the malicious traffic before it reaches its destination.

Think of an IPS like a **security guard posted at a locked entrance**: it evaluates traffic in real time and physically blocks unauthorized or dangerous individuals from entering.

### IDS vs. IPS

| Feature | IDS | IPS |
| :--- | :--- | :--- |
| **Traffic Monitoring** | Yes | Yes |
| **Alert Generation** | Yes | Yes |
| **Packet Blocking** | No | Yes |
| **Deployment Mode** | Passive (Out-of-band) | Active (In-line) |

### Industry-Standard IDS/IPS Solutions

When selecting an engine for a network environment, three major names dominate the open-source landscape:

#### 1. Snort

Developed by Martin Roesch and maintained by Cisco, Snort is one of the oldest, most robust, and most widely deployed IDS/IPS engines in the world.

- Rule-based detection
- Large community support
- Often used in enterprise environments
- Can operate in IDS or IPS mode

#### 2. Suricata

Suricata is a modern, high-performance IDS/IPS engine.

Key features:

- Multi-threaded architecture (faster than Snort in many cases)
- Supports IDS and IPS modes
- Built-in support for:
  - HTTP logging
  - TLS inspection
  - JSON-based logging (EVE JSON)
- Better suited for high-throughput environments

#### 3. Zeek (formerly Bro)

Zeek shifts away from traditional signature matching. Rather than checking packets against a list of known exploits, Zeek acts as a behavioral network security monitor.

- Focuses on traffic analysis, not signatures
- Generates rich logs of network activity
- Excellent for forensic analysis and SOC environments

#### Why We Choose Suricata for This Lab

We chose **Suricata** because:

- Perfect balance between raw performance, usability, and visibility.
- It integrates directly into pfSense
- Supports both IDS and IPS modes
- Uses ET Open rules (free and powerful)
- Provides structured logs (EVE JSON)
- Works well with SIEM tools like Graylog, Grafana, and Wazuh
- Can detect real-world attack patterns (Nmap, SQL injection, Metasploit, etc.)

### How Suricata Detects Threats

Suricata uses a combination of several advanced analytical methodologies to identify malicious activity:

- **Signature-Based Detection:** This is the core engine mechanism. Suricata compares incoming packets against thousands of predefined rules (signatures) that outline known attack patterns.
- Example: Nmap scan signatures, SQL injection strings

- **Anomaly-Based Detection:** By monitoring baseline network trends, Suricata can find deviations from normal behavior.
- Example: unusual traffic spikes or port scanning

- **Protocol Analysis:** Suricata understands application-layer protocols natively (e.g., HTTP, DNS, TLS, SMB). This allows it to spot malformed packets, invalid headers, or standard protocols being used on non-standard ports.

- **Behavioral Tracking:** This allows Suricata to observe activities over time.
- Example: repeated login failures or scanning behavior

### Decoding Suricata Rules

Rules serve as the underlying brain of Suricata. A rule defines:

- What traffic to inspect
- What patterns to match
- What action to take (alert, drop, log)

```text
ALERT tcp $EXTERNAL_NET any -> $HTTP_SERVERS 80 (msg:"ATTACK SQL Injection Attempt"; content:"UNION SELECT"; sid:1000001;)
```

If a packet originating from the outside network targeting an internal web server contains the string `UNION SELECT`, Suricata triggers an alert labeled with that specific message and ID.

In this lab, we use the **ET Open Ruleset**, a community-driven repository that provides continuously updated coverage for:

- Scan detection
- Exploits
- Web attacks
- Malware traffic
- Botnet communication
- DNS anomalies

### Lab Architecture Overview

Building on our previous lab setup, our virtual environment is hosted inside Proxmox VE. We have configured our network into functional VLANs to keep management traffic isolated from our vulnerable targets. All inter-VLAN and external traffic must route through our pfSense virtual firewall. By installing Suricata on pfSense, we create a strategic security checkpoint that forces all traffic to be deeply inspected before it can reach our internal network zones.

### Step-by-Step Deployment Guide

#### Step 1: Access the pfSense Dashboard

Begin by opening our web browser and authenticating into the web-based user interface of our pfSense firewall.

![pfSense dashboard](/assets/img/blog13/1-pfsense-dashboard.webp)
*Figure 1: Accessing the central pfSense dashboard interface*

#### Step 2: Install Suricata via the Package Manager

Navigate to **System > Package Manager** in the top navigation bar. Click on the **Available Packages** tab, search for `Suricata`, and locate the official entry.

![Suricata package manager](/assets/img/blog13/2-suricata-package-manager.webp)
*Figure 2: Searching for Suricata in the pfSense package repository*

Click **Install**, and confirm the action. The pfSense package system will download, unpack, and install all necessary Suricata binaries and dependencies automatically.

![Installing Suricata](/assets/img/blog13/3-installing-suricata.webp)
*Figure 3: Tracking the execution of the Suricata installation script*

Once finalized, you will see a successful installation confirmation. Suricata is now ready to configure under **Services > Suricata**.

![Installation complete](/assets/img/blog13/4-suricata-install-done.webp)
*Figure 4: Verifying the successful completion of the installation*

#### Step 3: Global Suricata Configuration

Navigate to **Services > Suricata** and open the **Global Settings** tab. This menu controls global behaviors, including which subscription feeds to use, how rules update, and general logging rules.

![Global settings](/assets/img/blog13/5-suricata-gloabl-settings.webp)
*Figure 5: Tweaking the global configuration parameters*

#### Step 4: Selecting Rule Feeds

Scroll down to the rule installations section. For our testing purposes, we select:

- ✅ ET Open Rules (primary)
- ❌ ET Pro (paid, not required)
- ❌ Snort rules (not needed for this lab)

![Rule selection](/assets/img/blog13/6-rules-types-to-install.webp)
*Figure 6: Enabling the open-source ET Open signature rulesets*

#### Step 5: Automating Rule Updates

Threat landscapes change daily. To ensure our signature base stays updated, configure the **Update Interval**. Setting this to update every 12 hours, a healthy balance between up-to-date threat feeds and efficient resource utilization.

![Rule update settings](/assets/img/blog13/7-rule-update-settings.webp)
*Figure 7: Scheduling automatic rule synchronization*

#### Step 6: Setting General Retention Preferences

Configure the basic system-level configurations below, ensuring that rule persistence is enabled. This ensures our customized settings and rule adjustments survive system restarts or automated firewall updates.

![General settings](/assets/img/blog13/8-general-settings-and-notifications.webp)
*Figure 8: Applying persistence and notification parameters*

#### Step 7: Executing the Initial Rule Download

With the feeds selected, navigate to the **Updates** tab. Click the **Update** button to manually force the initial signature download. pfSense will fetch, extract, and compile thousands of defensive signatures into our local database.

![Update rules](/assets/img/blog13/9-update-rules.webp)
*Figure 9: Fetching and parsing the community signatures for the first time*

#### Step 8: Configuring the Interface (VLAN 20)

Switch over to the **Suricata Interfaces** tab and add an interface. In this lab, we target **VLAN 20**, which hosts our vulnerable target infrastructure. Under the interface settings, enable logging and ensure that **EVE JSON Logging** is turned on. This is critical, as structured JSON data formats make it easy to parse log events out to an external SIEM later.

![Logging settings](/assets/img/blog13/10-vlan2-general-logging-settings.webp)
*Figure 10: Mapping Suricata to monitor our target network interface*

![EVE JSON output](/assets/img/blog13/11-vlan2-eve-output-settings.webp)
*Figure 11: Configuring structured EVE JSON output for logging depth*

Scroll down to performance tuning. Ensure the detection engine configurations match our hardware resources, balancing processing threads with available RAM allocations.

![Detection engine settings](/assets/img/blog13/12-vlan2-alert-performance-and-detection-engine-settings.webp)
*Figure 12: Optimization adjustments for the multi-threaded detection engine*

#### Step 9: Defining our Network Scope

For an IDS to accurately distinguish an inbound attack from an outbound data leak, it must understand what networks it protects. Under the variable configurations, define our **HOME_NET** (representing our internal VLAN ranges) and our **EXTERNAL_NET** (representing everything lying outside our trusted network perimeter) or simply:

- HOME_NET = internal VLAN networks
- EXTERNAL_NET = everything outside

![Network settings](/assets/img/blog13/13-vlan2-networks-suricata-should-inspect-alert-suppression-arguments-settings.webp)
*Figure 13: Standardizing structural variable assignments for HOME_NET*

![HOME_NET addresses](/assets/img/blog13/14-home-net-addresses.webp)
*Figure 14: Verifying local subnet masks allocated to the protection zone*

#### Step 10: Activating Rule Categories

Go to the **Categories** tab for our newly created interface. Select the specific signature categories you want to enforce. For this lab, make sure to enable categories covering reconnaissance scans, web server exploits, malware communications, and active OS vulnerabilities.

![Default rules](/assets/img/blog13/15-default-enabled-rules.webp)
*Figure 15: Reviewing the baseline enabled signature matrices*

From the ET Open Rules, I selected a focused set of rules that align with the systems and attack scenarios used throughout this lab. The goal was to detect reconnaissance, exploitation attempts, web application attacks, and common post-exploitation activity while minimizing unnecessary noise and false positives.

#### **Core Detection Rules**

These rules provide the foundation for detecting reconnaissance and exploitation activity.

- `emerging-scan.rules`
- `emerging-exploit.rules`
- `emerging-shellcode.rules`
- `emerging-attack_response.rules`

#### What These Rules Help Detect

- Nmap scans
- Service enumeration
- Metasploit exploitation attempts
- Reverse shells
- Suspicious payload delivery

#### **Web Application Security Rules**

Since this lab includes DVWA and OWASP Juice Shop, web application protection is particularly important.

- `emerging-web_server.rules`
- `emerging-web_client.rules`
- `emerging-web_specific_apps.rules`
- `emerging-sql.rules`

#### What These Rules Help Detect

- SQL Injection
- Cross-Site Scripting (XSS)
- Directory Traversal
- Command Injection
- Web Enumeration
- OWASP Top 10 attack techniques

#### **Malware & Compromise Detection**

These rules focus on identifying indicators of compromise and post-exploitation activity.

- `emerging-malware.rules`
- `emerging-compromised.rules`
- `emerging-remote_access.rules`

#### What These Rules Help Detect

- Malware communications
- Reverse shells
- Command-and-control (C2) traffic
- Remote access tools
- Post-exploitation activity

#### **Threat Intelligence & Behavioral Detection**

Not all attacks rely on known exploits. Some tools can be identified through their behavior, fingerprints, or communication patterns.

- `emerging-user_agents.rules`
- `emerging-ja3.rules`
- `emerging-dns.rules`

#### What These Rules Help Detect

- Nikto scans
- Nmap NSE scripts
- Burp Suite traffic patterns
- Unusual HTTP clients
- Suspicious DNS requests
- TLS fingerprint anomalies

![Custom rules set 1](/assets/img/blog13/16-custome-enabled-rules-set1.webp)
*Figure 16: Turning on specialized network scan and exploit signatures*

![Custom rules set 2](/assets/img/blog13/17-custom-enabled-rules-set2.webp)
*Figure 17: Fine-tuning explicit web attack and application layer categories*

#### Why These Rules Were Chosen

The selected rule categories directly support the attack scenarios performed later in this lab. As we launch Nmap scans, Nikto scans, and web application attacks from Kali Linux, Suricata will use these rules to identify reconnaissance activity, exploitation attempts, malicious payloads, and abnormal application behavior. This focused approach provides meaningful alerts while keeping the overall rule set manageable and easy to understand for beginners.

#### Step 11: Launching the Suricata Engine

Return to the main **Suricata Interfaces** view. Click the green **Play/Start** button next to our configured interface. The engine will read the settings, spin up its parsing threads, and change to an active status.

![Start Suricata](/assets/img/blog13/18-start-suricata.webp)
*Figure 18: Initializing the engine daemon on the interface*

![Suricata running](/assets/img/blog13/19-enabled-suricata.webp)
*Figure 19: Confirming that Suricata is actively parsing live traffic*

### Security Simulation and Alert Verification

To evaluate if our IDS works properly, we launched an attack machine from outside the network and ran aggressive scanning tools (`nmap`) alongside web vulnerability exploits against our hosted targets (DVWA, Juice Shop, and Metasploitable).

#### Step 12: Attack Simulation

With Suricata fully configured and actively monitoring the server VLAN, the next step is to validate that the IDS can successfully detect suspicious and potentially malicious activity. Before generating any attack traffic, it is useful to establish a baseline and verify that the monitored interface is operating normally.

![Before performing attacks](/assets/img/blog13/20-before-attacks.webp)
*Figure 20: Suricata monitoring the network before any attack traffic is generated.*

At this stage, the alert dashboard contains little to no suspicious activity, providing a clean baseline from which we can observe the effects of our testing. Establishing a baseline is an important step in security monitoring because it allows analysts to distinguish between normal network behavior and events that require investigation.

#### Performing Network Reconnaissance with Nmap

One of the first actions performed by attackers after gaining access to a network is reconnaissance. The objective is to identify live hosts, discover open ports, determine running services, and gather information about potential attack targets. Using Kali Linux, an Nmap service version scan was executed against systems located within the server VLAN:

```bash
nmap -sV 10.10.20.x
```

The `-sV` option instructs Nmap to probe discovered services and attempt to identify their versions. This information can help an attacker determine whether a system is vulnerable to known exploits.

![Nmap service version scan output](/assets/img/blog13/21-nmap-sv-scan-output.webp)
*Figure 21: Nmap service version scan identifying open ports and running services on target systems.*

From Suricata's perspective, this activity appears as reconnaissance behavior. The IDS is capable of detecting common scanning techniques and generating alerts that indicate port scanning, service enumeration, and other suspicious discovery activities. These early-stage detections are particularly valuable because they can provide warning signs before an attacker attempts exploitation.

#### Web Application Enumeration with Nikto

After identifying available web services, attackers commonly perform web application reconnaissance to discover vulnerabilities and misconfigurations. To simulate this stage of an attack, the Nikto web vulnerability scanner was used against one of the vulnerable web applications hosted in the lab environment.

```bash
nikto -h http://10.10.20.x
```

Nikto performs a wide range of security checks, including searches for dangerous files, exposed administrative interfaces, insecure configurations, outdated software versions, and known vulnerabilities.

![Nikto scan output](/assets/img/blog13/22-nikto-scan-output.webp)
*Figure 22: Nikto web vulnerability scan performing enumeration and security checks against a target web application.*

As Nikto generates numerous HTTP requests and probes various application components, Suricata can identify many of these actions as suspicious web activity. Depending on the enabled rule categories, alerts may be generated for abnormal user agents, web application reconnaissance, directory enumeration, exploit attempts, and other behaviors commonly associated with vulnerability assessment tools.

Together, the Nmap and Nikto scans demonstrate how Suricata provides visibility into the reconnaissance phase of an attack. Rather than waiting for a successful compromise, the IDS can identify and alert on the preparatory activities that attackers perform while gathering information about their targets.

#### Step 13: Reviewing the Alert Log

Navigate to the **Alerts** tab inside Suricata. Here, we can see live alerts popping up as the attacks hit our targets.

![After scan](/assets/img/blog13/23-scan-after-attack.webp)
*Figure 23: Real-time alert logs displaying captured attack traffic*

#### Step 14: In-Depth Log Analysis

Looking closely at the logged alerts under **Logs View**, we can dissect individual entries.

![Logs view](/assets/img/blog13/24-logs-view.webp)
*Figure 24: Analyzing log data details within the pfSense diagnostic UI*

Suricata successfully identified and logged:

- **Nmap Network Discovery Scans:** Noted by TCP packets with unusual flag combinations or fast port connections.
- **Web App Exploitation Patterns:** Caught directory traversal attempts and SQL Injection strings directed at DVWA and Juice Shop.
- **Metasploitable Exploits:** Tripped rules associated with known remote code execution (RCE) patterns and reverse-shell payloads.

### Project Results & Key Takeaways

Through this lab simulation, Suricata successfully monitored traffic passively and generated clear alerts for every stage of our attack simulation from:

- Nmap reconnaissance scans
- Web application attacks on DVWA and Juice Shop
- Exploitation attempts against Metasploitable systems
- Malicious HTTP patterns and payloads

### Core Technical Lessons Learned

- **Rule Management Matters:** Turning on every single rule category will quickly slow down our system and bury our console in false positives. Fine-tuning our rule selection to match only what actually lives inside our network is critical.
- **Strategic Placement is Key:** Placing Suricata directly on our pfSense gateway gives us a central vantage point to monitor all cross-VLAN traffic.
- **The Value of Structured Logging:** Enabling EVE JSON logging gives us clean, standardized data logs, which are essential for feeding events into a centralized SIEM system.
- **Hardware Limitations:** Real-time deep packet inspection demands significant CPU and RAM resources. When deploying an IDS on bare-metal hardware, utilizing network interface cards (NICs) that support hardware checksum offloading makes a huge difference in performance.

## Conclusion

This project highlights how a professional-grade network security monitoring architecture can be deployed in a home lab using Proxmox VE, pfSense, and Suricata. By intentionally launching real-world exploits against vulnerable targets in a controlled environment, we move past theory concepts to see exactly how attack patterns look on the wire.

Building this architecture provides practical hands-on experience with:

- Implementing network monitoring solutions at a central gateway.
- Tuning multi-threaded signature inspection engines to balance performance and coverage.
- Dissecting adversary behavior, from initial port scans to full application-layer exploits.
- Organizing structured network security logs for better visibility.

Setting up Suricata as an IDS is a massive first step toward securing a network, but it is only part of a complete defense-in-depth strategy. Now that our core monitoring engine is running and logging properly, our next developmental steps will focus on forwarding Suricata's structured EVE JSON logs into a centralized SIEM platform like Wazuh or an Elastic Stack. This will allow us to build comprehensive analytics dashboards, correlate network alerts with host-level system events, and establish real-world Security Operations Center (SOC) monitoring capabilities.

Happy Hacking and Stay Secure!!
