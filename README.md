# Cybersecurity-SOAR-EDR-Automation-Project-Lab

## Objective


The Detection Lab project aimed to establish a controlled environment for simulating and detecting cyber attacks. The primary focus was to ingest and analyze logs within a Security Information and Event Management (SIEM) system, generating test telemetry to mimic real-world attack scenarios. This hands-on experience was designed to deepen understanding of network security, attack patterns, and defensive strategies.

### Skills Learned


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
### Deploying the Endpoint VM in Vultr
![Cloud VM Endpoint Deployment](https://imgur.com/q027G7S.png)

To begin building out the SOAR–EDR lab environment, I deployed a dedicated cloud-based Windows endpoint using Vultr. I provisioned a compute instance and configured it as **MyDFIR-SOAR-EDR**, which serves as the primary workstation for simulating attacker activity and validating detection and response workflows.

I selected a Windows image for the operating system and chose a U.S. region (Miami) to maintain low network latency during testing. After configuring the instance, I launched it and verified that it was running successfully. This endpoint will later be equipped with the necessary EDR agent and logging configurations so it can forward telemetry to the rest of the SOAR–EDR pipeline.

Using a cloud-hosted endpoint provides a controlled and isolated environment where I can safely run commands, generate test incidents, and observe how detections move through the system—from the endpoint to the SIEM and ultimately into the automated SOAR playbooks built later in the project.

---
### Configuring the Firewall for the SOAR–EDR Endpoint
![Firewall Deployment](https://imgur.com/W39UMYh.png)

To secure the cloud-hosted endpoint, I created a dedicated firewall group in Vultr and applied a set of inbound rules designed to limit exposure while still allowing the access I needed for remote administration and testing. I named the group **SOAR-EDR-Firewall** and linked it directly to the Windows instance used in this project.

I configured the firewall to explicitly allow only two types of inbound traffic:

- **RDP (TCP 3389)** for managing the Windows endpoint.
- **SSH (TCP 22)** for situations where secure remote shell access is needed.

Everything else is blocked by default, with a final rule that drops any unspecified inbound traffic. This approach keeps the attack surface small and ensures the instance is only reachable through the intended management channels.

Setting up the firewall early in the project helped ensure that the environment stays isolated and protected as I build out the rest of the SOAR–EDR pipeline.

---
### Setting Up the LimaCharlie Organization and Installation Key
![LimaCharlie Organization](https://imgur.com/AhQ60J6.png)

To begin integrating endpoint telemetry into the SOAR–EDR pipeline, I created a dedicated organization in LimaCharlie named **Ken-SOAR-EDR**. This organization acts as the central workspace for managing sensors, collecting event data, and configuring detection and response capabilities.

After creating the organization, I generated an **Installation Key**, which is required for securely enrolling endpoints into the LimaCharlie platform. This key allows the Windows VM in my lab environment to register as a managed sensor, enabling it to send real-time telemetry such as process events, network activity, and security signals back to the cloud.

By setting up the organization and installation key early, I established the foundation for connecting the endpoint to the broader detection and automation workflow that supports the SOAR–EDR environment.
---
### Installing the LimaCharlie Agent on the Endpoint
![LimaCharlie installation](https://imgur.com/emaZReb.png)

With the organization and installation key configured in LimaCharlie, the next step was to install the LimaCharlie agent on the Windows endpoint. I used PowerShell to execute the installation command provided by the platform, which securely registered the endpoint with my **Ken-SOAR-EDR** organization.

Once the installer ran, the agent successfully initialized and connected to the LimaCharlie cloud, confirming that the endpoint was now actively reporting telemetry data. This process established the communication channel needed for collecting real-time event logs, process creation data, and other system-level activities directly from the VM.

By completing this installation, I effectively linked my cloud-based endpoint to the detection and response ecosystem—enabling the future stages of the project, such as alerting, incident investigation, and automated SOAR playbook execution.



