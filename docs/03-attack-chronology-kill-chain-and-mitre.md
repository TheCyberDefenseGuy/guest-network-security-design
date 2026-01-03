# Attack Chronology – Kill Chain and MITRE ATT&CK

## MITRE ATT&CK – TA0043: Reconnaissance
MITRE: TA0043  
Techniques:
- Network Service Discovery (T1046)
- Wireless SSID Discovery

**Reconnaissance** is the tactic where adversaries gather information about a target to support planning and execution of future attacks. This phase occurs before initial access and helps attackers reduce uncertainty and improve the effectiveness of subsequent techniques.

Reconnaissance activities may be **passive**, relying on publicly available information, or **active**, involving direct interaction with the target environment.

Common objectives during reconnaissance include:
- Identifying organizational structure, employees, and roles
- Collecting email addresses, domain names, IP ranges, and network details
- Researching public-facing applications and exposed services
- Gathering personal or professional information from open sources
- Mapping technologies, security controls, and potential weaknesses

The primary goal of this tactic is to enable more targeted, efficient, and successful attacks in later stages of the adversary lifecycle.

---

## MITRE ATT&CK – TA0001: Initial Access
MITRE: TA0001  
Techniques:
- Valid Guest association
- Portal abuse

**Initial Access** refers to the techniques adversaries use to gain an initial foothold within a target environment. This tactic represents the first direct interaction between the attacker and the victim’s systems.

Attackers commonly exploit weaknesses in technology, configuration, or human behavior to achieve initial access. Successful execution of this phase allows adversaries to establish persistence and proceed with further malicious activities.

Common initial access methods include:
- Phishing and other social engineering techniques
- Exploiting public-facing applications or services
- Valid account abuse (stolen or compromised credentials)
- Drive-by compromise and malicious file delivery
- Supply chain compromises

The objective of **Initial Access** is to breach the target environment while minimizing detection, enabling the attacker to continue the attack lifecycle.

---

## MITRE ATT&CK – TA0008: Lateral Movement
MITRE: TA0008  
Techniques:
- ARP spoofing (T1557)
- Multicast abuse
- Client-to-client scans

**Lateral Movement** refers to the techniques adversaries use to move through a compromised environment to access additional systems and accounts. After gaining an initial foothold, attackers leverage lateral movement to expand control and reach high-value assets.

This tactic often involves abusing legitimate credentials, remote services, and trust relationships within the network. Effective lateral movement enables attackers to escalate their impact while avoiding detection.

Common lateral movement techniques include:
- Remote services such as RDP, SMB, SSH, or WinRM
- Credential reuse and pass-the-hash or pass-the-ticket attacks
- Exploiting trust relationships between systems
- Remote execution of malicious payloads
- Using administrative tools and shared resources

The primary objective of **Lateral Movement** is to gain broader access to the environment and position the adversary to achieve their ultimate goals, such as data exfiltration or system compromise.

---

## MITRE ATT&CK – TA0009: Collection
MITRE: TA0009  
Techniques:
- Traffic capture
- Metadata analysis

**Collection** consists of techniques adversaries use to gather data of interest from compromised systems and environments. This data may include sensitive files, credentials, communications, or other valuable information.

During this phase, attackers identify, locate, and extract relevant data while attempting to remain stealthy and avoid detection. Collected information is often staged for later exfiltration or used to support further attack objectives.

Common collection activities include:
- Capturing files, databases, and documents
- Collecting credentials, keystrokes, or clipboard data
- Accessing email, chat logs, or browser data
- Taking screenshots or recording user activity
- Gathering data from network shares or cloud storage

The goal of **Collection** is to obtain information that supports espionage, financial gain, or operational objectives before moving to exfiltration or impact phases.

---

## MITRE ATT&CK – TA0040: Impact
MITRE: TA0040  
Techniques:
- Network DoS (T1498)
- Resource exhaustion

**Impact** refers to techniques adversaries use to disrupt, manipulate, or destroy systems, data, and operations. This tactic represents actions taken to achieve the attacker’s final objectives, which may include financial gain, operational disruption, sabotage, or reputational damage.

Adversaries may intentionally impair system availability, compromise data integrity, or interfere with business processes. These actions can have immediate and long-term consequences for the targeted organization.

Common impact techniques include:
- Data destruction or encryption (e.g., ransomware)
- Service disruption or denial-of-service attacks
- Defacement of websites or systems
- Manipulation or deletion of critical data
- Inhibiting system recovery or backups

The primary objective of **Impact** is to degrade the availability, integrity, or reliability of systems and data in order to fulfill the adversary’s mission.

---

## Key observation

Guest networks compress multiple MITRE tactics into **a single trust zone**, increasing risk density.
