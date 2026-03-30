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
*(200–400 words response here)*

### **3. DHCP Starvation & Rogue DHCP for Long-Term Persistence**  
*(200–400 words response here)*

### **4. VLAN Hopping & Subverting Network Segmentation**  
*(200–400 words response here)*

### **5. Wireless Attacks: Rogue AP vs. Evil Twin**  
*(200–400 words response here)*

### **6. IP Spoofing in Multi-Stage Attacks**  
*(200–400 words response here)*

### **7. DNS Cache Poisoning: Evolution of Attacks**  
*(200–400 words response here)*

### **8. BGP Hijacking: Attackers as Network Operators**  
*(200–400 words response here)*

### **9. Amplification DDoS Attacks: DNS vs. NTP vs. Memcached**  
*(200–400 words response here)*

### **10. DDoS Mitigation: Proactive vs. Reactive Defense**  
*(200–400 words response here)*

### **11. Emerging Cyber Threats in Cloud & AI-Driven Networks**  
*(200–400 words response here)*

### **12. Shaping Your Security Mindset**  
*(200–400 words response here)*

### **13. Designing a Secure Network: VLAN Segmentation & Access Control**  
*(200–400 words response here)*

### **14. Protecting Against DDoS & Global Threats**  
*(200–400 words response here)*

### **15. LAN Security: Preventing Internal Threats & Lateral Movement**  
*(200–400 words response here)*

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
2. How can it be prevented?

---

## **Attack 2: SYN Flood (With and Without IP Spoofing)**

### **Deliverables**
#### **Screenshots**
- [Insert required screenshots]

#### **Response to Analysis Questions**  
**Answer the following (400–600 words total):**
1. How does the attack pattern differ in Wireshark?
2. Why is IP spoofing effective for hiding attackers and bypassing defenses?
3. Which version of the attack would be harder to block with traditional firewalls?

---

## **Attack 3: Exploiting a Vulnerable Service (Remote Code Execution – RCE)**

### **Deliverables**
#### **Screenshots**
- [Insert required screenshots]

#### **Response to Analysis Questions**  
**Answer the following (400–600 words total):**
1. Why was the vsftpd 2.3.4 service vulnerable?
2. How did the exploit work?
3. What security measures could prevent this type of attack?

---

## **Attack 4: Passive LAN Sniffing & Reconnaissance with Wireshark**

### **Deliverables**
#### **Screenshots**
- [Insert required screenshots]

#### **Response to Analysis Questions**  
**Answer the following (400–600 words total):**
1. What did the attacker observe?
2. How could an organization prevent passive sniffing?
3. Why is HTTPS important?
