---
layout: post
title: "Chapter 10 - SSH Brute-Force Detection Using Wazuh"
date: 2025-12-28 12:00:00 +1100
authors: [avinash, angela]
categories: [homelab,siem]
tags: [wazuh, attack, detection]
description: Simulating and detecting an SSH password brute-force attack in Wazuh.
image: headers/01.jpg
media_subpath: /assets/img/posts/010-SSHBruteForceDetection/
---

## Introduction
In this chapter, we simulate an SSH password brute-force attack against a wazuh monitored endpoint. This helps demonstrate how security events are captured, analyzed, and presented through the Wazuh dashboard in real time.

---
## Prerequisites
- Wazuh agent installed on the victim machine (refer to [Chapter 9]({% post_url 2025-11-05-009-DeployingWazuhAgent %})).
- Wazuh server up and running.
- An attack machine (preferably Kali Linux).

---
## Simulating the Attack
1. Log in to the Kali Linux attack machine.
2. Use **Hydra** to launch an SSH brute-force attack against the target system:

```bash
hydra -l <USERNAME> -P <PASSWD_LIST.txt> ssh://<IP>
```

   Example:
```bash 
hydra -l user -P /usr/share/wordlists/metasploit/unix_passwords.txt ssh://192.168.1.146
```

![](001.png)

3. Once the attack is triggered, navigate to the **Wazuh Dashboard**. You will see multiple alerts generated in response to the repeated authentication failures.
![](002.png)

4. Go to **Threat Hunting** in the Wazuh dashboard. Here, you can observe a high number of authentication failure events (in this case, 278), indicating a brute-force attempt.
![](003.png)

5. Inspect individual threat hunting events. One of the events shows the description:  
   **“sshd: brute force trying to get access to the system. Authentication failed.”**  
   **Rule ID:** 5763 
![](004.png)

6. Expand the event to view detailed information such as source IP, attempted usernames, timestamps, and rule metadata.
![](005.png)
![](006.png)

---
## Conclusion
This chapter demonstrated how an SSH brute-force attack is detected and logged by Wazuh. The generated alerts and threat hunting events confirm that Wazuh effectively monitors authentication activity and provides detailed visibility into attempted attacks.

---
## Up Next
In the next chapter, we will use XDR capabilities to respond to and mitigate the simulated attack.