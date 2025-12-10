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


