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

---
### Endpoint Successfully Reporting to LimaCharlie
![sensor_list](https://imgur.com/AcPxKWb.png)

After deploying the Windows VM and installing the LimaCharlie agent, the endpoint began reporting live telemetry to my LimaCharlie organization. The screenshot shows the VM (`mydfir-soar-edr`) appearing under the **Sensors List**, confirming that the agent is authenticated, healthy, and actively streaming event data into the platform.

Seeing the sensor come online validated that the connection between the VM and the LimaCharlie cloud was fully established. This meant the environment was now ready for monitoring, threat detection, and automated response workflows. With the sensor live, I could start collecting real-time system activity, issuing remote commands, and building out the SOAR/EDR pipeline for the rest of the project.

---
### Credential Extraction & EDR Visibility Testing Using LaZagne
![laZagne_execution](https://imgur.com/5MGQ1nR.png)

To validate that my EDR pipeline could detect real-world credential harvesting behavior, I executed **LaZagne** inside my Windows analysis VM using PowerShell. LaZagne is a post-exploitation tool commonly used by attackers to recover stored credentials from browsers, the system registry, DPAPI, and other local sources.

After downloading the executable, I ran it directly from the VM’s `Downloads` directory. The screenshot shows LaZagne successfully enumerating the system’s masterkeys, decrypting DPAPI secrets, and dumping password hashes for local accounts such as `Administrator`, `Guest`, and `DefaultAccount`. It also extracted LSA secrets and other credential artifacts stored in Windows.

This step served two purposes in my SOAR–EDR workflow:

1. **Simulating a realistic credential-theft scenario**  
   By running an actual credential enumeration tool, I generated telemetry that mimics early-stage attacker activity. This allowed me to observe how the endpoint agent reports events such as process execution, command-line arguments, and file access patterns.

2. **Testing my detection and response pipeline**  
   With the agent connected to my EDR backend, I could verify that suspicious activity—like credential dumping—was being logged and was available for automated or manual response actions.

Overall, this helped confirm that the environment could correctly surface high-value security signals, giving me a solid foundation for building detections, alerts, and SOAR playbooks around credential-theft techniques.

---
### Endpoint Telemetry Captured in LimaCharlie After Executing LaZagne
![LimaCharlie_laZagne_events](https://imgur.com/DQaIGfh.png)

After running LaZagne on my Windows endpoint to generate realistic credential-theft activity, I moved into LimaCharlie to verify that the EDR agent was correctly capturing and forwarding telemetry. The screenshot above shows the event timeline filtered for activity related to the LaZagne execution.

LimaCharlie recorded a full chain of process events, including:
- The initial launch of `LaZagne.exe`
- Its child process activity (e.g., calls to `cmd.exe`)
- File access events as the tool attempted to read credential-related system files
- Hash and signature metadata for each process involved

What stood out in this capture is the clear parent/child relationship between the unsigned `LaZagne.exe` binary and the subsequent system processes it spawned. The event details panel shows the command-line arguments, file paths, cryptographic hashes, and signature status, which are exactly the indicators analysts rely on when investigating suspicious behavior.

This step allowed me to confirm that:
1. **The endpoint agent was operating correctly**  
   All process activity tied to LaZagne was logged with granular detail.

2. **The telemetry included enough context for detection engineering**  
   With signed/unsigned file indicators, parent-process lineage, and hash values, I was able to trace the execution flow end-to-end.

3. **The environment could support automated response workflows**  
   Since LimaCharlie exposes detection-grade event data, I could start building rules and SOAR actions to flag or contain credential-dumping activity.

Overall, this validated that my EDR pipeline is able to surface the type of high-value behavioral events that defenders need when dealing with credential-harvesting techniques used by real attackers.

---
### Custom Detection & Response Rule for LaZagne Activity
![Limacharlie_D&R_rule](https://imgur.com/zx1ISdT.png)

To extend the visibility I already had in LimaCharlie, I created a custom Detection & Response (D&R) rule focused on identifying credential-dumping behavior on my Windows endpoint. Since LaZagne is a well-known tool used by attackers for credential extraction, I used its execution as a practical example of how to build targeted detections in an EDR environment.

The screenshot above shows the final rule I implemented. The detection logic monitors both new and existing process events and applies a set of conditions to catch activity tied to LaZagne. I designed it to work specifically on Windows hosts and to trigger when the EDR agent observes a process whose file path or command line ends with `LaZagne.exe`. This ensures the rule picks up both direct execution and rename

---
### Creating a Dedicated Slack Workspace for SOAR Alerts

![Slack_SOAR-EDR_workspace](https://imgur.com/sB1C1c9.png)
To centralize the alerts coming out of my SOAR-EDR environment, I set up a dedicated Slack workspace specifically for this project. I wanted a clean space where I could route detections, automation outputs, and investigation notes without mixing them into my personal Slack channels.

The screenshot shows the workspace I created, named **MyDFIR-SOAR-EDR**. This workspace acts as the communication hub for the project. Once the workspace was up, I prepared it to receive webhook-based notifications from LimaCharlie so that any high priority detections, like credential dumping activity or suspicious process execution, would immediately appear in Slack.  

Having a separate workspace also made it easier to organize channels for different parts of the lab, such as detection alerts, automation logs, and general coordination. This setup mirrors how many teams manage communication during real security operations and helped build a more realistic workflow for the project.

---
### Creating a Dedicated Alert Channel in Slack
![Slack_alert Channel](https://imgur.com/YdRmpSM.png)
To support real-time security operations, I created a dedicated **#alerts** channel within the Slack workspace used for this SOAR-EDR project. The goal was to have a single, focused location where detection and response notifications could be delivered without noise from unrelated conversations.

The screenshot shows the **#alerts** channel immediately after creation. This channel was intentionally set aside for automated security alerts generated by the detection and response pipeline. By separating alerts from general discussion channels, it becomes much easier to triage incidents, spot high-severity activity quickly, and maintain an audit trail of security events.

This setup mirrors how many SOC teams structure their communication, using Slack as a lightweight incident notification system alongside their EDR and SOAR tools. With the channel in place, the environment was ready to receive automated alerts tied to endpoint detections and response actions.

---

### Creating an Automation Story in Tines
![Tine_story](https://imgur.com/X1CZxw5.png)
After setting up the SOAR tooling for this project, I created a dedicated **Story** in Tines to serve as the automation layer for security detections and responses. The screenshot shows the initial story titled **“MyDFIR-SOAR Project”**, which acts as the central workflow container for this environment.

In Tines, a story represents the full lifecycle of an automated security process, from receiving alerts, to enrichment, to triggering downstream actions such as notifications or response steps. Creating this story established a structured framework for designing and managing automation logic tied to endpoint detections from the EDR platform.

At this stage, the story was enabled and ready to be expanded with actions and decision logic as alerts are generated. This mirrors how SOAR platforms are commonly used in real SOC environments: starting with a core workflow and iteratively adding integrations, conditions, and response steps as detections mature

---
### Integrating LimaCharlie with Tines for Automated Response

![Tine&Limacharlie_integration](https://imgur.com/my61cym.png)
To enable automated incident response, I configured an **Output integration** in LimaCharlie that streams detection events directly into Tines. The screenshot shows the output named **MyDFIR-SOAR-EDR**, which is configured to forward events from the **detect** stream to Tines in real time.

This connection allows security detections generated by LimaCharlie, such as suspicious process execution or credential access activity, to be immediately consumed by Tines workflows. Instead of relying on manual review, alerts can now trigger automated actions like enrichment, conditional logic, and notifications without human intervention.

Establishing this integration effectively links endpoint detection with orchestration and response, creating a functional SOAR pipeline. With this in place, any high confidence detection can flow from the EDR platform into automation workflows designed to reduce response time and improve consistency in incident handling.

---
### Configuring Slack Credentials for Automated Alerting

![Slack_credentials](https://imgur.com/VED2RXi.png)
To enable real-time alert notifications, I configured a **Slack credential** within Tines that allows workflows to securely send messages to a dedicated alert channel. The screenshot shows the Slack credential stored alongside other integrations and managed directly within the Tines credential vault.

This setup uses Slack’s managed authentication, ensuring sensitive tokens are handled securely and not hardcoded into any workflow logic. Once configured, the credential can be reused across multiple actions, such as posting alerts, updating threads, or sending follow-up messages during an investigation.

By integrating Slack into the SOAR pipeline, security detections processed by Tines can be immediately delivered to analysts in a centralized communication channel. This removes the need to constantly monitor dashboards and enables faster awareness and response when suspicious activity is detected.

This configuration plays a critical role in closing the loop between detection and response, allowing automated workflows to notify the right people at the right time with actionable security context.

---
### Expanding Alert Delivery with Email Notifications

![Email](https://imgur.com/h3h3ede.png)
After establishing Slack-based alerting, I extended the workflow by adding an **email notification action** to ensure detections are delivered through multiple communication channels. The screenshot shows a single detection source triggering both Slack and email actions in parallel.

The workflow begins with a webhook that retrieves detection data from the EDR platform. Once a detection is received, the workflow branches to send a formatted alert message to a Slack channel while simultaneously dispatching an email notification. This design ensures alerts are not missed if an analyst is offline or away from Slack.

Email notifications provide redundancy and broader visibility, especially for high-priority detections that require immediate attention. By delivering alerts through multiple channels, the workflow improves reliability and reduces the risk of delayed response.

This approach reflects real-world SOC practices, where automated detections are surfaced across different tools to support timely triage and escalation.

---
### Adding Analyst Decision Control with a User Prompt
![user_prompt](https://imgur.com/zWTGeV6.png)

To move beyond passive alerting, I enhanced the workflow by adding a **user prompt** that introduces analyst decision-making into the response process. Instead of automatically taking action on every detection, the workflow now pauses and requests human confirmation before proceeding.

When a detection is retrieved through the webhook, the workflow presents a structured prompt to the analyst containing key context, including the alert title, timestamp, affected host, source IP, username, file path, command line, sensor ID, and a direct link to the detection. This information allows the analyst to quickly assess the situation without switching tools.

The prompt asks whether the affected endpoint should be isolated, giving the analyst a clear **Yes or No** decision. This approach mirrors real SOC workflows, where containment actions are validated by a human to reduce the risk of disrupting legitimate activity.

By integrating a user prompt into the automation, the workflow balances speed and control—automating data collection and notification while keeping high-impact response actions under analyst oversight.

---
### Handling Analyst “No” Decisions in the Response Workflow

![user_prompt](https://imgur.com/OdUQEX8.png)

I implemented a conditional branch in the workflow to handle cases where the analyst decides **not** to isolate the endpoint. When the user prompt returns a **“No”** response, the workflow follows a separate path that avoids taking any containment action on the host.

Instead of isolating the machine, this branch sends a message to Slack documenting the analyst’s decision. The notification confirms that the alert was reviewed and that isolation was intentionally declined, helping maintain visibility and accountability within the security team.

This design ensures that high-impact actions are never taken blindly. It preserves normal business operations while still capturing decision context and maintaining an audit trail of how alerts were handled. The workflow reflects real-world SOC practices where not every detection warrants containment, but every decision should be tracked and communicated.

---
## Automated Endpoint Isolation (Analyst-Approved Response)

In this part of the lab, I built a response path that runs when an analyst chooses **“Yes”** at the decision prompt. This workflow ties together **LimaCharlie (EDR)**, **Tines (SOAR)**, and **Slack** to automate containment while keeping a human in the loop.

When **“Yes”** is selected, Tines sends an **HTTP request to LimaCharlie’s API** to isolate the affected endpoint (sensor). This immediately limits the system’s network access, helping contain the threat and reduce the risk of lateral movement. After issuing the isolation command, the workflow sends another HTTP request to **check the isolation status**, making sure the action was successfully applied.

Once the status is confirmed, Tines sends a message to **Slack** with the outcome, giving the security team real-time visibility into what happened and confirming that the endpoint is now contained.

This setup mirrors how many SOC teams operate in real environments: **detections from an EDR trigger a SOAR workflow, an analyst approves the action, and automation handles the technical response**, with Slack used for clear and immediate communication.



