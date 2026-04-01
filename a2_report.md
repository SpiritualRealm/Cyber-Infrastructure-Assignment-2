### **Assignment 2 Report**  
#### CSCY 4743: Cyber and Infrastructure Defense, Spring 2026

**Name & Student ID**: John Paul Bennett Jr., 110412273  

---

# **Section 1: Conceptual Assignments**

### **1. ARP Poisoning & Advanced MITM Techniques**  
Attackers have developed several ways to bypass traditional ARP defenses such as static ARP entries and Dynamic ARP Inspection (DAI). One method is **MAC spoofing combined with VLAN hopping**, where an attacker impersonates a trusted device and gains access to a protected VLAN. Another approach involves targeting the control plane—for example, exploiting misconfigured switches or flooding the switch’s CAM table (CAM table overflow), forcing it into a hub-like behavior and enabling packet sniffing. Attackers may also exploit weaknesses in DAI deployments, such as untrusted ports incorrectly marked as trusted or gaps in DHCP snooping bindings, allowing forged ARP replies to pass validation.

Proxy ARP manipulation further enhances an attacker’s ability to intercept traffic. In proxy ARP, a device responds to ARP requests on behalf of another host. An attacker can abuse this by falsely advertising themselves as the gateway or another key node, even across subnet boundaries. This allows them to redirect traffic not just within a local subnet, but between subnets, effectively expanding the scope of a Man-in-the-Middle (MITM) attack. By positioning themselves as a “legitimate” intermediary, attackers can silently inspect, modify, or drop packets.

Even in a fully switched network with VLAN segmentation, ARP-based MITM attacks are still possible under certain conditions. If an attacker gains access to a specific VLAN, they can still poison ARP caches within that VLAN. Additionally, misconfigurations such as improperly isolated VLANs, trunk ports exposed to end devices, or lack of private VLANs can allow lateral movement. Techniques like VLAN hopping (via double tagging or switch spoofing) may let an attacker reach other VLANs. Once inside, they can perform ARP poisoning locally. Furthermore, if inter-VLAN routing relies on shared gateways, compromising ARP resolution for that gateway enables interception of traffic flowing between VLANs.


### **2. ARP Spoofing in IPv6 Networks**  
In IPv6 networks, Address Resolution Protocol (ARP) is replaced by the Neighbor Discovery Protocol (NDP), which handles tasks like address resolution and router discovery. To perform a man-in-the-middle (MITM) attack similar to ARP poisoning, an attacker uses NDP spoofing. This involves sending forged Neighbor Advertisement (NA) or Router Advertisement (RA) messages to trick devices into associating the attacker’s MAC address with a legitimate IPv6 address (such as a default gateway). As a result, traffic intended for another device is redirected through the attacker, enabling interception, modification, or disruption of communications.

In an NDP spoofing attack, the attacker typically floods the network with fake NA messages, claiming ownership of a target IPv6 address. Alternatively, by sending malicious RA messages, the attacker can convince hosts to use a rogue router under their control. Tools like THC-IPv6 can automate these attacks, making them more practical in real-world scenarios. Comparing NDP spoofing to ARP poisoning, there are notable differences in difficulty and effectiveness. ARP poisoning in IPv4 networks is relatively simple because ARP lacks authentication and is widely supported by straightforward attack tools. NDP spoofing, on the other hand, is somewhat more complex due to IPv6’s additional features, such as the potential use of Secure Neighbor Discovery (SEND), which introduces cryptographic protections. However, SEND is rarely implemented in practice, leaving many IPv6 networks vulnerable. In terms of effectiveness, both attacks can achieve similar MITM outcomes. However, NDP spoofing can be more powerful because it can also manipulate router advertisements, allowing attackers to influence network configuration (e.g., default gateways, DNS settings). This broader scope can make IPv6 attacks potentially more impactful, even if they require slightly more expertise to execute.



### **3. DHCP Starvation & Rogue DHCP for Long-Term Persistence**  
**DHCP Starvation as DoS:**
DHCP starvation is effective because it exploits the limited pool of available IP addresses on a network. An attacker floods the DHCP server with a large number of spoofed DHCP discovery/request messages, each using a different fake MAC address. The server responds by allocating IP leases until the pool is exhausted. Legitimate clients are then unable to obtain an IP address, effectively denying them network access. Because DHCP is typically unauthenticated and operates automatically, this attack is easy to execute and can disrupt entire subnets with relatively low effort.

**Rogue DHCP for MITM and Persistence:**
After exhausting the legitimate DHCP server, an attacker can introduce a rogue DHCP server to respond to new client requests. This allows them to supply malicious configuration parameters such as a fake default gateway, DNS server, or NTP server. While this enables MITM attacks (intercepting or redirecting traffic), it can also provide long-term persistence. For example, the attacker can assign controlled DNS servers to redirect users to phishing sites or maintain traffic visibility over time. By setting long lease durations, the attacker ensures clients retain these malicious settings even after the attack infrastructure is temporarily removed, creating a stealthy foothold that persists across reboots and user sessions.

**Defense-in-Depth Mitigation:**
A layered approach significantly reduces the risk:

* **DHCP Snooping:** Switches monitor DHCP traffic and only allow responses from trusted ports (where legitimate DHCP servers reside). This blocks rogue DHCP servers from issuing leases.
* **802.1X Authentication:** Ensures that only authenticated devices can connect to the network. This prevents unauthorized attackers from gaining the initial access needed to launch starvation or rogue DHCP attacks.
* **VLAN Segmentation:** Limits broadcast domains and isolates network segments. Even if an attack occurs, its impact is confined to a smaller portion of the network, reducing overall disruption and preventing lateral spread.

Together, these controls create overlapping protections—preventing unauthorized access, blocking malicious DHCP behavior, and containing potential damage—making rogue DHCP attacks far less effective.


### **4. VLAN Hopping & Subverting Network Segmentation**  
**Bypassing VLAN Segmentation (beyond classic attacks):**
Attackers have several additional avenues to circumvent VLAN isolation. Misconfigured **trunk ports** are a common weakness—if a switch port is left in trunk mode or allows unnecessary VLANs, an attacker can craft tagged frames to access restricted segments. Exploiting **native VLAN mismatches** between switches can also leak traffic across boundaries. Another method involves abusing **Layer 2 control protocols** (e.g., STP manipulation) to reposition the attacker logically within the network path, enabling traffic interception. Compromising a device that legitimately participates in multiple VLANs (like a router, firewall, or hypervisor with virtual switches) can also provide a pivot point to traverse segments. Additionally, weaknesses in **ACL enforcement** or inter-VLAN routing policies can allow unintended access even if VLAN separation exists at Layer 2.

**Insider Threat & Misconfiguration Abuse:**
An insider with limited access can escalate privileges by identifying poorly secured switch ports or misassigned VLAN memberships. For example, if unused ports are left active or assigned to sensitive VLANs, simply plugging into them grants elevated access. Insiders might also reconfigure their system to mimic trusted devices (MAC spoofing) or exploit weak port security settings. If management interfaces (e.g., switch admin panels) are exposed on user VLANs, an insider could alter VLAN assignments or trunk configurations directly. Even small configuration oversights—like overly permissive inter-VLAN routing—can allow lateral movement into restricted departments (e.g., finance or HR networks).

**Exploiting Dynamic VLAN Assignment (RADIUS):**
Dynamic VLAN assignment via RADIUS (often tied to 802.1X) can be subverted if authentication mechanisms are weak. An attacker might steal valid credentials (phishing or credential reuse) to authenticate as a higher-privileged user and be placed into a sensitive VLAN. If fallback mechanisms exist (e.g., MAC Authentication Bypass), attackers can spoof MAC addresses of authorized devices to gain similar access. Misconfigured RADIUS policies—such as overly broad role assignments—can also grant unintended VLAN access. In some cases, insufficient validation of RADIUS responses or lack of mutual authentication between network devices and the RADIUS server can allow attackers to inject or relay authentication messages, resulting in unauthorized VLAN placement.

Overall, VLAN security depends heavily on correct configuration, strict access control, and layered defenses; missteps in any of these areas can undermine segmentation.


### **5. Wireless Attacks: Rogue AP vs. Evil Twin**  
**Bypassing MAC Filtering & SSID Hiding:**
MAC filtering and hidden SSIDs provide minimal real security because both rely on observable information. Wireless traffic exposes client MAC addresses in cleartext, so an attacker can passively capture frames, identify an authorized device, and perform **MAC spoofing** to impersonate it. SSID hiding is also ineffective: the network name is still revealed in association and probe requests from legitimate clients. Tools that sniff wireless traffic can easily recover the SSID, allowing an attacker to join or target the network despite these “security” measures.

**Deauthentication + Rogue AP for Session Hijacking:**
An attacker can escalate from eavesdropping to active compromise by combining deauthentication attacks with a rogue access point (AP). First, they send forged deauthentication frames to disconnect clients from the legitimate AP. Then, they stand up a rogue AP (often mimicking the same SSID and signal strength—an “evil twin”). Clients automatically reconnect to the stronger or first-available signal, which is now the attacker’s AP. Once connected, the attacker can perform MITM techniques such as traffic inspection, SSL stripping (if protections are weak), or credential harvesting via captive portals. Over time, this can evolve into full session hijacking by stealing cookies, tokens, or authentication headers, especially on poorly secured or non-encrypted services.

**WPA3 SAE (Dragonfly) Improvements & New Risks:**
WPA3 replaces the WPA2 Pre-Shared Key handshake with the Simultaneous Authentication of Equals (SAE), based on the Dragonfly protocol. SAE prevents offline dictionary attacks by ensuring that password verification requires active interaction with the access point; captured handshakes cannot be brute-forced offline. Each authentication attempt involves ephemeral key exchange, so attackers cannot validate guesses without engaging the network directly, significantly slowing attacks.

However, new risks exist. Early implementations were vulnerable to side-channel attacks (e.g., timing or cache-based leaks, sometimes referred to as “Dragonblood”), which could reveal information about the password. Additionally, downgrade attacks may force clients to fall back to WPA2 if transition modes are enabled. While WPA3 is more secure by design, misconfigurations and implementation flaws can still introduce exploitable weaknesses.


### **6. IP Spoofing in Multi-Stage Attacks**  
**IP Spoofing & Session Hijacking (UDP vs TCP):**
IP spoofing allows an attacker to forge the source IP address of packets, making them appear to come from a trusted host. In session hijacking, this enables the attacker to inject malicious traffic that the victim system believes is legitimate. This technique is especially effective with **UDP-based protocols** because UDP is connectionless and lacks built-in mechanisms for authentication, sequencing, or acknowledgment. There is no handshake or state tracking, so if an attacker can guess the correct destination and port, spoofed packets are often accepted without validation. This makes attacks like DNS spoofing or streaming/session injection relatively straightforward.

In contrast, **TCP** includes a three-way handshake, sequence numbers, and acknowledgments, making blind spoofing more difficult. An attacker must correctly predict or observe sequence numbers to successfully inject packets into an active session. Without this information, spoofed packets are typically rejected.

**Why TCP Session Hijacking Is Still Possible:**
Despite modern defenses like randomized initial sequence numbers (ISNs), full TCP session hijacking remains possible under certain conditions. First, attackers can perform **on-path (MITM)** attacks—through techniques like ARP/NDP spoofing or rogue access points—allowing them to observe and synchronize with legitimate sequence and acknowledgment numbers in real time. Once visibility is achieved, injecting packets becomes trivial.

Second, **side-channel attacks** can sometimes infer sequence numbers indirectly, even when they are randomized. For example, timing differences, IP ID field behavior, or shared resource leakage can help attackers narrow down valid sequence ranges.

Third, **session desynchronization attacks** can disrupt one endpoint (e.g., via packet flooding or reset injection), allowing the attacker to take over the session while the legitimate client is effectively silenced.

Finally, poorly implemented TCP stacks or legacy systems may still use predictable sequence patterns, making them vulnerable. Thus, while sequence randomization increases difficulty, it does not eliminate risk—especially when attackers can combine spoofing with traffic interception or other weaknesses.


### **7. DNS Cache Poisoning: Evolution of Attacks**  
**Pre-Kaminsky vs. Kaminsky-style attacks:**
Before the Kaminsky DNS cache poisoning attack, DNS cache poisoning was relatively limited. Attackers had to guess both the transaction ID (TXID) and the correct response timing for an existing query—making success difficult and often requiring predictable resolver behavior. The Kaminsky technique fundamentally changed this by triggering large numbers of DNS queries for random subdomains (e.g., `abc123.victim.com`). Each query created a new opportunity to inject a forged response, dramatically increasing the probability of guessing the correct TXID.

Mitigations included **randomized query IDs** and **source port randomization**. Instead of a 16-bit TXID alone, resolvers also randomize the UDP source port, effectively expanding the entropy attackers must guess (from ~2¹⁶ to ~2³² possibilities). This makes blind spoofing attacks far less feasible at scale.

**DNSSEC Adoption Challenges:**
DNSSEC ensures DNS integrity through cryptographic signatures, but adoption has been slow due to operational complexity. Managing key pairs (ZSKs/KSKs), performing secure key rollovers, and maintaining proper chain-of-trust configurations introduce significant administrative overhead. Misconfigurations can cause entire domains to become unreachable. Additionally, DNSSEC increases response sizes, leading to fragmentation issues and requiring infrastructure upgrades. Many organizations also perceive limited immediate benefit compared to the effort required, especially when other protections (like HTTPS) are already in place.

**Bypassing DNSSEC Protections:**
Even with DNSSEC, attackers can still succeed by targeting weaker points in the ecosystem. If a **DNS resolver is compromised**, it can return malicious but seemingly valid responses to clients, bypassing validation entirely. Similarly, attackers can manipulate **upstream infrastructure**—such as intercepting traffic between resolvers and authoritative servers via BGP hijacking or MITM attacks—forcing resolvers to accept downgraded (non-DNSSEC) responses if validation is not strictly enforced. In some environments, “DNSSEC optional” configurations allow fallback to insecure DNS, creating an opportunity for downgrade attacks.

Thus, while DNSSEC strengthens authenticity and integrity, its effectiveness depends heavily on correct deployment, strict validation, and securing the broader DNS infrastructure.


### **8. BGP Hijacking: Attackers as Network Operators**  
**Why BGP lacks authentication & route leaks vs. hijacks:**
The Border Gateway Protocol was designed in an era where the internet was smaller and built on mutual trust between operators. As a result, it lacks built-in authentication and strong validation of route announcements—routers generally trust that peers advertise legitimate IP prefixes.

A **route leak** occurs when a network unintentionally announces routes learned from one peer to another peer in violation of routing policies (often due to misconfiguration). Traffic may take inefficient or unintended paths, but the goal is not malicious. In contrast, a **BGP hijack** is intentional: an attacker (or misbehaving AS) falsely advertises ownership of IP prefixes they do not control, diverting traffic to themselves for interception, manipulation, or blackholing.

**Nation-state exploitation:**
A nation-state can leverage BGP manipulation for both censorship and intelligence gathering. By hijacking or selectively announcing prefixes, they can **reroute traffic through infrastructure they control**, enabling large-scale surveillance (deep packet inspection, metadata collection). They can also **blackhole or throttle traffic** to suppress access to specific services or foreign content. More subtly, they can perform **traffic shaping or interception** without detection by briefly rerouting data and then forwarding it to the legitimate destination, maintaining normal user experience while collecting intelligence.

**2018 Route 53 Hijack & Defenses:**
The Amazon Route 53 BGP hijack involved attackers redirecting traffic intended for Amazon’s DNS service to malicious servers, enabling cryptocurrency theft. Several defenses could have mitigated or prevented the attack:

* **RPKI (Resource Public Key Infrastructure):** Cryptographically verifies that an AS is authorized to announce specific IP prefixes. Invalid announcements would be rejected.
* **Route filtering & prefix validation:** ISPs should enforce strict filtering of customer advertisements to ensure only legitimate prefixes are propagated.
* **BGP monitoring & anomaly detection:** Real-time systems can detect unusual route changes and trigger rapid mitigation.
* **BGPsec (emerging):** Extends BGP with path validation, though adoption is still limited.

These mechanisms introduce verification and accountability into BGP, reducing reliance on trust and making large-scale hijacks significantly harder.


### **9. Amplification DDoS Attacks: DNS vs. NTP vs. Memcached**  
**Why Memcached amplification is more dangerous:**
Memcached–based amplification attacks are significantly more powerful than DNS or NTP reflection because of their extreme amplification factor. While DNS and NTP typically amplify traffic by tens or hundreds of times, exposed Memcached servers (especially over UDP) can amplify requests by **tens of thousands of times**. Attackers send small spoofed requests that trigger very large responses (cached data), overwhelming victims with massive traffic volumes. Additionally, many Memcached instances were historically exposed to the internet without authentication, making them easy to abuse at scale.

**Amplification via TCP despite UDP blocking:**
Blocking UDP reduces classic reflection attacks, but it does not eliminate amplification risks. Attackers can still exploit **TCP-based amplification vectors**. For example:

* **SYN floods with spoofed IPs** can exhaust server resources (state exhaustion).
* **Reflection via misconfigured TCP services** (e.g., HTTP, HTTPS, or proxies) can be abused using techniques like request smuggling or leveraging large asymmetric responses (e.g., small request → large file download).
* **Botnets** can replace spoofing entirely—infected devices send legitimate TCP requests to amplify traffic without needing forged IPs.

While TCP requires a handshake (making spoofing harder), attackers compensate using distributed botnets or exploiting services that generate disproportionate responses.

**BGP spoofing & global-scale DDoS:**
Through manipulation of Border Gateway Protocol, attackers can enhance DDoS attacks in several ways. By hijacking IP prefixes, they can **impersonate the victim’s IP space**, enabling more effective spoofing (since return paths are controlled). They can also **reroute global traffic flows**, directing large volumes of legitimate or reflected traffic toward a target. In some cases, attackers create **traffic blackholes or loops**, amplifying congestion across multiple networks.

At a larger scale, BGP manipulation allows attackers to position themselves strategically within internet routing paths, effectively turning parts of the global network into unwilling participants in a DDoS campaign. This combination of routing control and amplification techniques makes such attacks highly disruptive and difficult to mitigate without coordinated responses from multiple network operators.


### **10. DDoS Mitigation: Proactive vs. Reactive Defense**  
**Effective defenses against multi-vector DDoS:**
A resilient strategy combines **proactive scaling** with **real-time filtering**. Using **Anycast routing**, traffic is distributed across geographically dispersed nodes, absorbing volumetric floods close to their source and preventing a single choke point. At the edge, **rate limiting** and connection throttling help contain SYN floods, HTTP floods, and API abuse without overwhelming backend systems.

Layered **traffic scrubbing** (via cloud-based DDoS protection services) filters malicious packets before they reach the origin. These systems rely on **behavioral analysis** and anomaly detection to distinguish legitimate users from bots, adapting dynamically to new attack patterns. For application-layer attacks, **WAFs (Web Application Firewalls)** enforce rules, block suspicious payloads, and mitigate techniques like request flooding or cache bypassing.

Additionally, **autoscaling infrastructure** ensures services can elastically handle traffic spikes, while **CDNs** cache content and reduce origin load. Combining these with **protocol hardening** (e.g., SYN cookies, limiting amplification vectors) creates a defense-in-depth model that addresses volumetric, protocol, and application-layer attacks simultaneously.

**Zero-Trust Networking & DDoS Mitigation:**
Zero-trust shifts the mindset from “allow then filter” to **“never trust, always verify.”** Instead of exposing services broadly, access is tightly controlled based on identity, device posture, and context. This reduces the attack surface significantly—services are often hidden behind identity-aware proxies rather than publicly reachable endpoints.

In a DDoS context, zero-trust minimizes reliance on perimeter defenses alone. Unauthorized traffic is dropped early because it cannot authenticate, reducing the effectiveness of application-layer floods. It also enables **granular access controls**, ensuring only verified clients consume resources, which limits resource exhaustion attacks.

Moreover, zero-trust architectures integrate well with **continuous monitoring and behavioral analytics**, allowing faster detection of anomalies and automated mitigation. While it does not stop large volumetric floods by itself, it complements traditional defenses by ensuring that even if traffic reaches the network, only authenticated and policy-compliant requests are allowed to interact with critical services.


### **11. Emerging Cyber Threats in Cloud & AI-Driven Networks**  
**Emerging attack vectors in cloud, AI, and edge environments:**
Cloud-native and AI-driven systems introduce new, less-visible attack surfaces. In **serverless architectures**, attackers target event triggers, misconfigured permissions, or abuse short-lived functions to execute malicious code without leaving persistent artifacts. Compromising **CI/CD pipelines** enables injection of backdoors into applications before deployment, creating large-scale supply chain risks.

At the edge, distributed devices expand the attack surface—compromised edge nodes can be used for **data poisoning**, traffic manipulation, or as entry points into centralized systems. AI-driven environments introduce **model-specific attacks**, such as adversarial inputs that manipulate outputs, or **training data poisoning** that subtly biases models over time. Attackers can also use AI themselves to automate reconnaissance, generate convincing phishing campaigns, or dynamically adapt malware to evade detection.

**Limits of traditional security models:**
Legacy network security relies heavily on **perimeter-based defenses** (firewalls, IDS/IPS), assuming clear boundaries between “trusted” internal networks and “untrusted” external ones. This model breaks down in cloud-native systems where workloads are ephemeral, distributed, and communicate over dynamic APIs rather than fixed network paths.

In **serverless computing**, there are no long-lived hosts to monitor, making traditional endpoint-based tools ineffective. Visibility is reduced because execution is abstracted by the cloud provider. Similarly, **AI-manipulated attacks** can evade signature-based detection since they generate novel, adaptive patterns rather than known indicators of compromise.

Supply chain attacks further challenge traditional defenses because malicious code may originate from **trusted vendors or dependencies**, bypassing perimeter controls entirely. Once inside, these threats appear legitimate.

To address these gaps, organizations must adopt **identity-centric security**, continuous monitoring, runtime protection, and strict least-privilege access controls. Integrating security directly into development pipelines (DevSecOps) and leveraging AI for defense are increasingly necessary to keep pace with evolving threats in modern, highly dynamic environments.


### **12. Shaping Your Security Mindset**  
Studying both local (LAN) and global (WAN) attacks highlights that network defense is no longer about just guarding the perimeter—security must be **layered, adaptive, and context-aware**. LAN attacks, like ARP/NDP spoofing, rogue DHCP, VLAN hopping, or insider threats, show that even an internal network assumed to be “trusted” can be compromised quickly. Meanwhile, WAN-level attacks, including BGP hijacks, global DDoS, or supply-chain compromises, illustrate how vulnerabilities at the broader internet scale can cascade into local networks.

This perspective makes it clear that **internal network visibility and segmentation** are as critical as perimeter defenses. Measures such as **DHCP snooping, 802.1X authentication, VLAN isolation, and internal traffic monitoring** are no longer optional—they prevent attackers from pivoting once inside. At the same time, proactive WAN protections like **Anycast DDoS mitigation, route filtering, and secure BGP practices** remain vital for keeping external threats from overwhelming services.

If I had to prioritize, I would focus first on **strengthening internal network defenses**. The reasoning is that perimeter security alone is insufficient: sophisticated attackers or insiders who bypass the WAN edge can exploit trust relationships within the LAN, making internal segmentation, access controls, and monitoring essential. Once internal security is robust, perimeter defenses can then act as a force multiplier, limiting exposure to global attacks and containing incidents before they spread internally.

Ultimately, the key takeaway is that **security mindset must shift from perimeter-only thinking to a defense-in-depth approach**, treating every network layer, every device, and even internal communications as potentially untrusted until verified. This balanced strategy reduces both the likelihood and impact of attacks across the entire organization.


### **13. Designing a Secure Network: VLAN Segmentation & Access Control**  
**Designing VLAN Segmentation to Minimize LAN Attacks:**
As a Security Administrator, I would implement **strict VLAN segmentation based on functional and security boundaries**. Critical departments like Finance, HR, and IT would each reside in separate VLANs, minimizing the potential lateral movement of an attacker exploiting ARP/NDP spoofing or VLAN hopping. **Private VLANs** or **port isolation** could further restrict communication between hosts within the same VLAN, while sensitive servers (e.g., domain controllers, databases) could be placed in dedicated management VLANs with controlled access. Additionally, **trunk ports** would only carry explicitly required VLANs, and **native VLANs** would be changed to an unused VLAN ID to prevent double-tagging exploits.

**Balancing Security and Operational Complexity:**
Segmentation must not excessively hinder business operations. I would use **role-based VLAN assignments** combined with **802.1X authentication and Dynamic VLAN Assignment via RADIUS** so devices automatically receive appropriate network access based on their identity. For example, IT workstations may need access to multiple VLANs for management tasks, while HR devices are restricted strictly to HR VLANs. Grouping less sensitive systems together in shared VLANs can reduce management overhead without exposing critical resources. Documentation, standardized templates, and automation tools help maintain consistency while limiting human error.

**Mitigating VLAN Misconfigurations and Access Risks:**
Common misconfigurations include:

* **Unused ports left in default VLANs**, which attackers could exploit for unauthorized access. Mitigation: disable unused ports or assign them to a “quarantine” VLAN.
* **Improper trunk port configuration**, allowing VLAN hopping or leakage. Mitigation: restrict allowed VLANs per trunk, change native VLAN, and use port security.
* **Weak or missing 802.1X policies**, enabling rogue devices to join VLANs. Mitigation: enforce strong authentication and MAC filtering with dynamic VLAN assignment.
* **Overly permissive inter-VLAN routing**, which can bypass segmentation. Mitigation: use firewall rules or ACLs to explicitly control cross-VLAN traffic.

By combining **principle of least privilege**, consistent configuration practices, and automated enforcement, VLAN segmentation can reduce the impact of LAN-based attacks while remaining operationally manageable across diverse departments.


### **14. Protecting Against DDoS & Global Threats**  
**Immediate Incident Response Steps:**
During a large-scale DDoS attack, the first priority is **maintaining service availability** while identifying and mitigating malicious traffic. Immediate steps include:

1. **Activate the incident response plan** and notify stakeholders.
2. **Analyze traffic patterns** using network monitoring tools to identify the attack type (volumetric, protocol-based, or application-layer).
3. **Implement traffic filtering or rate limiting** at the edge, blocking obvious attack sources or suspicious IP ranges.
4. **Engage upstream ISPs or cloud mitigation services** to divert or scrub traffic before it reaches the network.
5. **Preserve logs and telemetry** for post-incident analysis, ensuring forensic data is retained for root cause investigation.

**Long-Term DDoS Mitigation Strategies:**
To reduce the impact of future attacks, a **defense-in-depth approach** is essential:

* **Volumetric attacks:** Leverage Anycast routing and global scrubbing centers to distribute and absorb traffic.
* **Protocol attacks:** Harden network equipment with SYN cookies, connection limits, and protocol anomaly detection.
* **Application-layer attacks:** Deploy Web Application Firewalls (WAFs) and behavioral analytics to detect abnormal requests and block malicious patterns.
* **Capacity planning and autoscaling:** Ensure cloud and on-prem resources can elastically handle traffic spikes.
* **Continuous monitoring and simulation:** Regularly test defenses using DDoS drills or red-team exercises to identify weaknesses.

**Cloud vs. On-Premise Defenses:**

* **Cloud-based mitigation services** (e.g., AWS Shield, Cloudflare) provide rapid, scalable protection against large-scale attacks, especially volumetric floods that exceed on-premise capacity. They also offer global traffic scrubbing, reducing the risk of local network saturation.
* **On-premise defenses** are essential for low-latency filtering, protecting internal resources, and mitigating smaller protocol or application-layer attacks before traffic leaves the local network.
* **Hybrid approach:** Combining cloud and on-premise solutions provides layered protection—on-prem filtering reduces load and latency, while cloud services absorb massive traffic spikes and provide global reach.

This layered strategy ensures resilience, maintains service continuity, and allows rapid adaptation to evolving attack techniques.


### **15. LAN Security: Preventing Internal Threats & Lateral Movement**  
**Detecting and Containing a LAN Foothold:**
If an attacker establishes a presence inside the LAN, rapid detection and containment are critical to prevent lateral movement or privilege escalation. I would rely on **real-time network monitoring and anomaly detection**, looking for unusual ARP/NDP activity, unexpected DHCP leases, or traffic flows to unauthorized VLANs. Network segmentation with **firewalls and ACLs between VLANs** allows suspicious hosts to be quarantined quickly. Additionally, deploying **endpoint detection and response (EDR) agents** helps identify abnormal behavior at the device level, such as privilege escalation attempts or unusual process activity. Once detected, I would **isolate affected hosts** (e.g., via quarantined VLANs or switch port shutdowns) while investigating the breach, minimizing risk to other segments.

**Preventing Insider Threats and Rogue Devices:**
A layered combination of defenses is most effective:

* **Port Security:** Configure switches to allow only a limited number of MAC addresses per port and shut down or restrict ports that show violations, preventing unauthorized devices from connecting.
* **ARP/DHCP Protections:** Enable **DHCP snooping**, **dynamic ARP inspection**, and **NDP protections** to prevent spoofing, rogue DHCP servers, and man-in-the-middle attacks.
* **Authentication Mechanisms:** Implement **802.1X network access control**, optionally combined with **dynamic VLAN assignment via RADIUS**, so only authorized users and devices can join the network and are placed in the correct VLANs.

This combination prevents rogue devices from joining the LAN and limits the potential for insiders to misuse privileges.

**Enforcing Policies and Reducing Human Error:**
Even strong technical controls can fail if misconfigurations occur. To address this:

* **Automated configuration management** ensures switches, routers, and firewalls are consistently set according to policy.
* **Policy enforcement via templates and scripts** reduces the chance of manual errors.
* **Regular audits and continuous monitoring** detect deviations early.
* **Security training for IT staff and employees** reinforces best practices, like not plugging unauthorized devices or bypassing authentication controls.
* **Role-based access control** ensures users can only perform tasks necessary for their job, reducing accidental misconfigurations.

By combining technical controls, automation, monitoring, and human awareness, an organization can greatly reduce insider threats and the risk of lateral movement while keeping operations manageable.


---

# **Section 2: Practical Assignments — Network Attacks Simulation, Mitigation, and Analysis**

---

## **Attack 1: ARP Spoofing (Local MITM – Simplified Version)**

### **Deliverables**
#### **Screenshots**
- [Insert required screenshots]

#### **Response to Analysis Questions**  
**Answer the following (400–600 words total):**
1. What does ARP poisoning enable attackers to do?

Address Resolution Protocol (ARP) poisoning is a network attack technique that exploits the way devices on a local network map IP addresses to MAC (hardware) addresses. By sending falsified ARP messages, an attacker can associate their own MAC address with the IP address of another device, such as a router or another host. This manipulation allows the attacker to intercept, modify, or disrupt network traffic. One of the primary capabilities ARP poisoning enables is man-in-the-middle (MITM) attacks. In this scenario, the attacker positions themselves between two communicating devices—commonly a user’s computer and the network gateway. Once in this position, the attacker can eavesdrop on data being transmitted, capturing sensitive information such as login credentials, session cookies, or financial data. If the traffic is unencrypted (e.g., HTTP instead of HTTPS), the attacker can read it directly; even with encryption, they may attempt techniques like SSL stripping to downgrade security. Additionally, ARP poisoning allows attackers to modify data in transit. This means they can alter messages, inject malicious code, or redirect users to fake websites (phishing). For example, a victim attempting to visit a legitimate banking site could be redirected to a fraudulent page designed to steal credentials. Another consequence is denial of service (DoS). By poisoning ARP tables with incorrect mappings, the attacker can disrupt communication between devices, effectively preventing them from reaching each other or the internet. This can degrade network performance or completely halt connectivity. ARP poisoning can also be used as a stepping stone for more advanced attacks, such as session hijacking, where the attacker takes over an active session between a user and a service, gaining unauthorized access without needing login credentials.

2. How can it be prevented?

Preventing ARP poisoning involves a combination of technical controls and good network practices. One effective method is the use of static ARP entries, where critical devices (like servers and routers) have fixed IP-to-MAC mappings configured manually. Since these entries do not change dynamically, they are immune to spoofed ARP replies. However, this approach is not scalable for large or dynamic networks. Another important defense is network segmentation. By dividing a network into smaller subnets or VLANs, administrators can limit the scope of ARP traffic and reduce the number of devices an attacker can target. This containment makes it harder for attackers to perform widespread poisoning. The use of secure protocols is also critical. Encrypting traffic with protocols like HTTPS, SSH, or VPNs ensures that even if an attacker intercepts the data, it remains unreadable and less useful. While this does not prevent ARP poisoning itself, it significantly reduces its impact. Modern switches often support Dynamic ARP Inspection, a security feature that validates ARP packets against a trusted database (such as DHCP snooping tables). If an ARP packet contains invalid or suspicious information, it is dropped, preventing poisoning attempts. Finally, intrusion detection and prevention systems and specialized tools like ARP monitoring software can detect unusual ARP activity, such as frequent changes in IP-to-MAC mappings. These tools can alert administrators to potential attacks in real time. In summary, ARP poisoning is a powerful technique that enables attackers to intercept, manipulate, and disrupt network communications. Preventing it requires layered defenses, including static configurations, network design strategies, secure communication protocols, and active monitoring.


---

## **Attack 2: SYN Flood (With and Without IP Spoofing)**

### **Deliverables**
#### **Screenshots**
- [Insert required screenshots]

#### **Response to Analysis Questions**  
**Answer the following (400–600 words total):**
1. How does the attack pattern differ in Wireshark?

In Wireshark, different attack patterns can often be distinguished by how packets appear in terms of source/destination addresses, protocol behavior, and traffic consistency. For a normal connection-oriented attack such as a TCP SYN flood without spoofing, you typically see a large volume of SYN packets coming from a consistent or small set of source IP addresses, often followed by incomplete handshakes (missing the final ACK). The pattern looks repetitive and traceable, making it easier to correlate traffic to a specific origin. In contrast, when IP spoofing is used, the attack pattern becomes far more chaotic. The source IP field changes constantly, often appearing as random or widely distributed addresses. In Wireshark, this results in a flood of packets with little to no repetition in source identity, and often no successful handshake completion because responses are sent to nonexistent or uninvolved hosts. This randomness makes the traffic look more like a distributed attack, even if it originates from a single machine, and complicates analysis because there is no stable endpoint to follow.

2. Why is IP spoofing effective for hiding attackers and bypassing defenses?

IP spoofing is effective for hiding attackers and bypassing defenses primarily because it exploits the trust model of IP-based communication. Many systems and defenses rely on source IP addresses for identification, filtering, or rate limiting. By falsifying the source IP, an attacker can disguise their true origin, making traceback extremely difficult. Additionally, spoofing allows attackers to impersonate trusted hosts or generate traffic that appears to come from many different locations, overwhelming systems that are designed to handle limited connections per IP. It also enables reflection and amplification attacks, where responses are sent to a victim rather than the attacker, further obscuring the attacker's identity. Since the attacker does not need to receive responses, they can generate large volumes of traffic without maintaining a two-way connection, reducing the risk of detection. This asymmetry makes spoofing particularly powerful against systems that depend on verifying client legitimacy through IP reputation or connection state.

3. Which version of the attack would be harder to block with traditional firewalls?

The version of the attack that would be harder to block with traditional firewalls is the spoofed or distributed variant, especially when it mimics a distributed denial-of-service pattern. Traditional firewalls often rely on rules based on IP addresses, ports, and connection states. When an attack comes from a single or small set of identifiable IPs, these can be blocked or rate-limited effectively. However, when spoofing is used, blocking becomes impractical because each packet may appear to originate from a different address. Firewalls cannot easily distinguish between legitimate and malicious packets when both appear structurally valid and come from seemingly unrelated sources. Furthermore, stateless or poorly configured firewalls may not verify the legitimacy of TCP handshakes, allowing spoofed SYN packets to consume resources. Even stateful firewalls can be overwhelmed if the volume is high enough or if the attack exploits protocol weaknesses. As a result, spoofed attacks bypass simple filtering rules and require more advanced mitigation techniques such as ingress filtering, anomaly detection, or upstream traffic scrubbing, which go beyond traditional firewall capabilities.

---

## **Attack 3: Exploiting a Vulnerable Service (Remote Code Execution – RCE)**

### **Deliverables**
#### **Screenshots**
- [Insert required screenshots]

#### **Response to Analysis Questions**  
**Answer the following (400–600 words total):**
1. Why was the vsftpd 2.3.4 service vulnerable?

The vsftpd 2.3.4 service was vulnerable due to a compromised distribution rather than a traditional programming flaw. In 2011, attackers managed to tamper with the official source code hosted on a public download server and inserted a malicious backdoor into the software. This meant that system administrators who downloaded and installed that specific version unknowingly deployed a Trojanized service. The vulnerability was therefore a supply chain attack, where trust in the software’s origin was exploited. Since vsftpd (Very Secure FTP Daemon) was widely regarded as a secure and lightweight FTP server, many users did not suspect that the downloaded package had been altered. The compromised version still functioned normally as an FTP service, which helped conceal the malicious code and delayed detection.

2. How did the exploit work?

The exploit module exploit/unix/ftp/vsftpd_234_backdoor works by leveraging the hidden backdoor embedded in the compromised version. Instead of exploiting a memory corruption issue or authentication bypass, it simply triggers functionality that was intentionally planted by the attacker. When a client connects to the FTP server and sends a specially crafted username containing a smiley face sequence “:)”, the backdoor is activated. Upon receiving this input, the malicious code instructs the server to open a shell listener on TCP port 6200. The attacker can then connect to this port and gain command-line access to the system. This shell typically runs with the privileges of the FTP service, which may be elevated depending on system configuration. Because the exploit uses normal FTP communication to trigger the backdoor, it can bypass some basic security monitoring tools that only look for malformed packets or obvious intrusion patterns. The attack is simple, reliable, and does not require brute force or complex payload delivery, making it particularly dangerous in environments where the compromised version is present.

3. What security measures could prevent this type of attack?

To prevent this type of attack, organizations must adopt multiple layers of security focused on software integrity and system hardening. First and foremost, verifying the authenticity of software downloads is essential. Administrators should always validate cryptographic checksums (such as SHA256) or digital signatures provided by trusted developers to ensure that the file has not been altered. Using official repositories or package managers instead of manual downloads significantly reduces the risk of installing compromised software. Second, implementing the principle of least privilege can limit the impact of a breach. Running services like FTP under restricted user accounts or within containers prevents attackers from gaining full system control if a backdoor is exploited.

Additionally, network security controls play a key role. Firewalls should be configured to restrict unnecessary outbound and inbound connections, which would help detect or block unexpected behavior such as a service opening a shell on an unusual port like 6200. Intrusion detection and prevention systems (IDS/IPS) can also be tuned to flag anomalous activity, including unusual login patterns or unexpected port usage. Regular patch management and vulnerability scanning ensure that outdated or compromised software versions are quickly identified and replaced. Finally, maintaining detailed logs and actively monitoring them allows administrators to detect suspicious events early, reducing the dwell time of attackers and minimizing potential damage.

---

## **Attack 4: Passive LAN Sniffing & Reconnaissance with Wireshark**

### **Deliverables**
#### **Screenshots**
- [Insert required screenshots]

#### **Response to Analysis Questions**  
**Answer the following (400–600 words total):**
1. What did the attacker observe?

From the Wireshark capture, the attacker is passively observing unencrypted HTTP traffic between two hosts. Specifically, the attacker can clearly see a login transaction to the Mutillidae web application. The captured packet shows an HTTP POST request to `/mutillidae/index.php?page=login.php`, and in the packet details, the form data is fully visible. This includes sensitive credentials such as `username = "msfadmin"` and `password = "msfadmin"`. Because HTTP transmits data in plaintext, the attacker is able to read not only the request method and URL, but also the contents of the form submission, headers, and session-related data. In short, the attacker gains direct access to login credentials without needing to exploit the server—simply by listening to network traffic.

2. How could an organization prevent passive sniffing?

To prevent passive sniffing, organizations must focus on both network-level protections and secure communication protocols. One key defense is encryption. Using secure protocols like HTTPS (HTTP over TLS) ensures that even if packets are captured, their contents cannot be easily read. Network segmentation is another important measure: separating sensitive systems into different VLANs or subnets reduces the ability of an attacker to capture traffic. Additionally, switching from hubs to switches (or properly configuring switches) limits broadcast traffic visibility. Organizations can also deploy tools like intrusion detection systems (IDS) or intrusion prevention systems (IPS) to monitor for suspicious network activity, including sniffing attempts. Secure Wi-Fi configurations (WPA3 or at least WPA2 with strong passwords) help prevent attackers from joining the network in the first place. Finally, using VPNs for remote access ensures that traffic is encrypted end-to-end, even over untrusted networks.

3. Why is HTTPS important?

HTTPS is critical because it provides confidentiality, integrity, and authentication. First, confidentiality is achieved through encryption: data exchanged between the client and server is encrypted using TLS, preventing attackers from reading sensitive information such as usernames, passwords, or session cookies. In the scenario shown, if HTTPS had been used instead of HTTP, the attacker would not have been able to see the credentials in plaintext. Second, integrity ensures that the data cannot be modified in transit without detection. This protects against man-in-the-middle attacks where an attacker might attempt to alter requests or responses. Third, authentication ensures that the client is communicating with the legitimate server, verified through digital certificates issued by trusted certificate authorities.

Without HTTPS, any attacker with access to the same network—such as on public Wi-Fi or a compromised internal network—can easily capture and inspect traffic. This makes HTTP inherently insecure for transmitting sensitive data. HTTPS mitigates this risk by encrypting the communication channel, making passive sniffing attacks largely ineffective. Therefore, enforcing HTTPS across all web applications, especially those involving authentication or sensitive data, is a fundamental security best practice.
