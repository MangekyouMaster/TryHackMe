## I. Use Cases of Logs

Logs are vital records of activities, and they serve multiple critical functions across an organization.

| Use Case | Description |
| :--- | :--- |
| **Security Events Monitoring** | Logs help detect anomalous and malicious behavior when used in real-time monitoring systems. |
| **Incident Investigation & Forensics** | Logs are the traces of all activities, offering detailed information for **root cause analysis** and reconstruction of security incidents. |
| **Troubleshooting** | Logs record system and application errors, providing essential data to diagnose and fix technical issues. |
| **Performance Monitoring** | Logs offer valuable insights into the performance, usage, and efficiency of applications and systems. |
| **Auditing & Compliance** | Logs establish a trail of activities (**audit trail**), which is crucial for meeting regulatory and internal compliance requirements. |

---

## II. Types of Logs

Different activities are recorded in dedicated log files to maintain organization and relevance.

| Log Type | Usage | Example Events |
| :--- | :--- | :--- |
| **System Logs** | Troubleshooting running issues and providing information on various **Operating System (OS)** activities. | System Startup/Shutdown, Driver Loading, System Errors, Hardware events. |
| **Security Logs** | Detecting and investigating incidents related to system security and user access. | Authentication/Authorization events, Security Policy changes, User Account changes, Abnormal Activity. |
| **Application Logs** | Recording specific events related to the activity happening **inside an application**. | User Interaction, Application Changes/Updates, Application Errors. |
| **Audit Logs** | Providing detailed information on system changes and user events, helpful for **compliance** and security monitoring. | Data Access, System Change events, Policy Enforcement. |
| **Network Logs** | Providing information on the network’s **outgoing and incoming traffic** for troubleshooting and investigations. | Network Connection Logs, Network Firewall Logs, Traffic events. |
| **Access Logs** | Providing detailed information about **access to specific resources** (e.g., servers, databases). | Webserver Access Logs, Database Access Logs, API Access Logs. |

---

## III. Windows Event Logs Analysis

Windows OS organizes its logs into a few crucial categories, which can be viewed using the **Event Viewer** utility.

### Key Windows Log Types
* **Application:** Logs information, warnings, and errors related to applications running on the OS.
* **System:** Logs information related to OS operations, including driver issues, hardware issues, and service status.
* **Security:** The most critical log for security, recording all security-related activities like **user authentication** and security policy changes.

### Essential Security Event IDs
Windows events are identified by a unique **Event ID**. These IDs are crucial for security analysis.

| Event ID | Description |
| :--- | :--- |
| **4624** | A user account **successfully logged in** |
| **4625** | A user account **failed to log in** |
| **4634** | A user account successfully **logged off** |
| **4720** | A user account was **created** |
| **4724** | An attempt was made to **reset an account’s password** |
| **4722** | A user account was **enabled** |
| **4725** | A user account was **disabled** |
| **4726** | A user account was **deleted** |

### Event Viewer Utility
The **Event Viewer** provides a Graphical User Interface (GUI) to view, filter, and analyze these logs. Key fields in an individual log entry include:
* **Log Name:** The specific log file (e.g., Security, System).
* **Logged:** The time of the activity.
* **Event ID:** Unique identifier for the activity (as listed above).
* **Description:** Detailed information about the activity.

---

## IV. Web Server Access Logs Analysis (Linux/Apache)

Web servers log every request made to a website. These access logs are critical for security monitoring and understanding user traffic.

### Sample Access Log Fields
Web server logs (like those from Apache, often found in `/var/log/apache2/access.log`) typically contain the following key fields:

| Field | Example Value | Description |
| :--- | :--- | :--- |
| **IP Address** | `172.16.0.1` | The IP address of the user making the request. |
| **Timestamp** | `[06/Jun/2024:13:58:44]` | The time the request was made. |
| **HTTP Method** | `GET` | The action requested (e.g., `GET`, `POST`). |
| **URL** | `/` | The specific resource or path requested. |
| **Status Code** | `200` | The server's response code (e.g., `200` OK, `404` Not Found, `500` Server Error). |
| **User-Agent** | `Mozilla/5.0...` | Information about the user's OS, browser, etc. |

### Manual Log Analysis Commands (Linux)
Command-line utilities are essential for efficient log analysis in a Linux environment.

| Command | Usage | Purpose |
| :--- | :--- | :--- |
| **`cat`** | `cat access.log` | **Display entire file contents**. Can also combine files: `cat access1.log access2.log > combined_access.log` |
| **`grep`** | `grep "192.168.1.1" access.log` | **Search** for strings or patterns inside a log file and display matching lines. |
| **`less`** | `less access.log` | **View logs one page at a time**, helpful for large files. Use `spacebar` (next page) and `b` (previous page). |
| **Search in `less`** | `/pattern` then `n` | While in `less`, type `/` followed by a pattern (e.g., `/404`) to search. Use `n` (next) and `N` (previous) for occurrences. |
```
