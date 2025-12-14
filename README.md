# Cybersecurity-SOAR-EDR-Automation-Project-Lab

## Objective
[Brief Objective - Remove this afterwards]

The Detection Lab project aimed to establish a controlled environment for simulating and detecting cyber attacks. The primary focus was to ingest and analyze logs within a Security Information and Event Management (SIEM) system, generating test telemetry to mimic real-world attack scenarios. This hands-on experience was designed to deepen understanding of network security, attack patterns, and defensive strategies.

### Skills Learned
[Bullet Points - Remove this afterwards]

- Advanced understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting network logs.
- Ability to generate and recognize attack signatures and patterns.
- Enhanced knowledge of network protocols and security vulnerabilities.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used

- **LimaCharlie**:  Detection, telemetry, and endpoint isolation.

- **Tines**: SOAR automation platform.

- **Slack + Email**: Alerting and communication.

- **draw.io**:  Used to create the workflow diagram.
## Steps
![SOAR–EDR logical diagram ](https://imgur.com/amxuFf8.png)

This diagram represents the workflow I designed in Part 1 of the SOAR–EDR project. The goal of the project is to build hands-on experience with Endpoint Detection & Response (EDR) and Security Orchestration, Automation, and Response (SOAR) by integrating LimaCharlie (EDR) with Tines (SOAR) to create an automated incident-response playbook.
This workflow is my planning document before building the automation. It maps out each step the playbook will take after LimaCharlie detects suspicious activity—in this case, a “hack tool” execution on an endpoint.
## What the Playbook Does

 **1. Detection (LimaCharlie)**
When **LimaCharlie** detects malicious activity (such as a hack tool), it forwards the detection event to **Tines** for automated response handling.

 **2. SOAR Automation (Tines)**
Once **Tines** receives the alert, it automatically sends detailed notifications to:

- **Slack**
- **Email**

Each message includes key investigation details:

- Timestamp  
- Endpoint (computer) name  
- Source IP  
- Process & command line  
- File path  
- Sensor ID  
- Detection link (if available)

 **3. User Decision Point**
Tines then prompts the analyst with the question:

> **“Do you want to isolate this machine?”**

The workflow branches based on the analyst’s response.

 **If the user selects YES**
- Tines instructs LimaCharlie to **isolate the endpoint automatically**.  
- LimaCharlie performs the isolation.  
- Slack receives a confirmation message containing:
  - The isolation status  
  - A note stating the endpoint has been successfully isolated  

 **If the user selects NO**
- Tines sends a Slack message stating:
  - The endpoint **was NOT isolated**
  - A reminder to **investigate the activity further**
---
![Cloud VM Endpoint Deployment](https://imgur.com/q027G7S.png)

### Deploying the Endpoint VM in Vultr

To begin building out the SOAR–EDR lab environment, I deployed a dedicated cloud-based Windows endpoint using Vultr. I provisioned a compute instance and configured it as **MyDFIR-SOAR-EDR**, which serves as the primary workstation for simulating attacker activity and validating detection and response workflows.

I selected a Windows image for the operating system and chose a U.S. region (Miami) to maintain low network latency during testing. After configuring the instance, I launched it and verified that it was running successfully. This endpoint will later be equipped with the necessary EDR agent and logging configurations so it can forward telemetry to the rest of the SOAR–EDR pipeline.

Using a cloud-hosted endpoint provides a controlled and isolated environment where I can safely run commands, generate test incidents, and observe how detections move through the system—from the endpoint to the SIEM and ultimately into the automated SOAR playbooks built later in the project.

---
### Configuring the Firewall for the SOAR–EDR Endpoint

To secure the cloud-hosted endpoint, I created a dedicated firewall group in Vultr and applied a set of inbound rules designed to limit exposure while still allowing the access I needed for remote administration and testing. I named the group **SOAR-EDR-Firewall** and linked it directly to the Windows instance used in this project.

I configured the firewall to explicitly allow only two types of inbound traffic:

- **RDP (TCP 3389)** for managing the Windows endpoint.
- **SSH (TCP 22)** for situations where secure remote shell access is needed.

Everything else is blocked by default, with a final rule that drops any unspecified inbound traffic. This approach keeps the attack surface small and ensures the instance is only reachable through the intended management channels.

Setting up the firewall early in the project helped ensure that the environment stays isolated and protected as I build out the rest of the SOAR–EDR pipeline.


