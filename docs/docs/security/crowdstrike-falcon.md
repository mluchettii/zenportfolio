---
tags:
    - ANY.RUN
    - CrowdStrike Falcon
    - EDR
    - Linux
    - OSINT
    - Security
    - Triage
    - Virtualization
    - VirusTotal
    - Windows
---

# CrowdStrike Falcon

## Description

[**Falcon**](https://www.crowdstrike.com/en-us/platform/) is CrowdStrike's cloud-based endpoint security platform that provides next-generation antivirus (NGAV), endpoint detection and response (EDR), threat intelligence, and more capabilities to prevent all types of attacks. It installs on endpoint devices as a lightweight sensor that consumes minimal resources and can replace traditional antivirus and other independent security tools. Falcon supports Windows, Linux, macOS, and mobile devices.

## Falcon sensor

The **Falcon sensor** is a lightweight, intelligent software module installed on endpoints such as laptops, desktops, and servers. It operates in real time to monitor system activity and behavior, capturing and recording data on running processes, network connections, file access, and other endpoint events. This enables it to detect malicious activity quickly, block attacks on the system, and provide continuous endpoint detection and response (EDR). EDR is one of Falcon's core capabilities. It is best described as a solution that records behaviors at system-level for each endpoint, and it uses a multitude of data analytics techniques to detect suspicious behavior. Falcon also provides contextual information, blocks malicious activity, and provides suggestions for remediation of affected systems.

### Windows installation

1. Download the Falcon sensor from the Falcon Main view hub.

2. Paste in your **Customer ID** and click **Install**.

3. Once installation has finished, your device will appear in the hosts dashboard.

### Linux installation

1. From **Sensor downloads**, download and install the appropriate package for your distribution.


    ```bash title="For Debian-based"
    sudo apt install ~/Downloads/falcon-sensor_.deb
    ```

2. Post-install configuration:

    ```bash title="a. Enter the Customer ID to the Falcon configuration"
    sudo /opt/CrowdStrike/falconctl -s --cid=xxxxx
    ```

    ```bash title="b. Start and enable the falcon-sensor system daemon"
    sudo systemctl start falcon-sensor
    sudo systemctl enable falcon-sensor
    ```

    ```bash title="c. Verify that the Falcon sensor is running"
    sudo ps -e | grep falcon-sensor
    # Output should look like the following:
    # 3191 ?    00:00:05 falcon-sensor-b
    ```

3. The new Linux host should appear on your hosts dashboard.

## Next-gen antivirus

CrowdStrike's Falcon sensor is capable of detecting malware in real time using advanced techniques. This includes file analysis, machine learning, behavioral analysis, and threat intelligence -- all of these help Falcon quickly identify and respond to both known and unknown malware, ransomware, and fileless attacks. Once a threat is detected, Falcon isolates the affected endpoint and generates detailed reports for security teams, also providing tactical insights from frameworks like MITRE ATT&CK.

### Triage process

To gain an understanding of the triage process, prepare a Windows 11 virtual machine with the Falcon sensor installed on it. Then, practice threat detection and investigation with Falcon, threat analysis with **VirusTotal** and **ANY.RUN**, and then compile a triage report.

#### Threat detection

1. Install and configure the Falcon sensor on a Windows 11 virtual machine.

2. In the VM, download a Trojan from this GitHub repository:

    !!! warning "CAUTION: These files CAN ruin your computer! Always use them within a sandboxed environment."

        <https://github.com/Da2dalus/The-MALWARE-Repo>

3. If the file downloaded successfully, then the CrowdStrike Falcon sensor will instantly detect and quarantine the Trojan.

#### Threat investigation

1. From the Falcon Hub, click on the **Next-gen AV** tab. You will see a list of recently quarantined items with file names, date of quarantine, affected host, and the user associated with the event.

2. Expand the sidebar, then navigate to **Endpoint security -> Activity dashboard**.

3. Click **See details** beside the most recent detection.

    !!! note "Detection details"

        This information pertains to the event's severity, detection tactic and technique, process origin, etc. Take note of the SHA256 **hash value** found under the **Triggering Indicator**. This hash value is unique to the malicious file and can be used for further threat investigation.


#### Threat analysis

##### VirusTotal

1. Copy the hash value to your clipboard.

2. Go on [**VirusTotal**](https://www.virustotal.com/) and paste the hash into the **Search** field.

3. Here, you can see the severity score, security vendor analyses, behavior, community insights, etc., pertaining to the threat.

##### ANY.RUN

1. Go on [**ANY.RUN**](https://app.any.run/) to detonate the malware in an interactive malware analysis sandbox (business email required).

2. Click on **Submit URL** and enter the URL of the malware repo from earlier:

    <https://github.com/Da2dalus/The-MALWARE-Repo>

3. Upon starting the VM, a browser will automatically take you to the malware repo. Try downloading and detonating some of them.

4. Upon detonation, monitor the resource usage and process information on the right pane to see what happens to the machine in real time.

5. Click view more information to see advanced details of the process.

#### Triage documentation

##### Triage documentation template

Document your findings using the following template:

``` title="Triage documentation template"
Description:
Date:
Time Quarantined:
Source IP:
Destination IP:
User:
File Path:
Filenames:
SHA-1:
SHA-256:
MD5:
Analysis:

Status:
```

##### Triage documentation examples

Here are a couple of examples:

``` title="Triage A"
Description: msedge.exe on LENOVO by marcos
Date: Jun. 20, 2025
Time Quarantined: 10:57:39
Source IP: xx.xxx.xxx.xxx
Destination IP: xx.xxx.xxx.xxx
User: marcos
File Path: \Device\HarddiskVolume3\Users\marcos\Downloads\Zika.exe
Filenames: Zika.exe
SHA-1: 86165eb8eb3e99b6efa25426508a323be0e68a44
SHA-256: 1a904494bb7a21512af6013fe65745e7898cdd6fadac8cb58be04e02346ed95f
MD5: 40228458ca455d28e33951a2f3844209

Analysis: Malicious executable file (Zika.exe) detected on LENOVO host's
user "marcos" with IP address of xx.xxx.xxx.xxx. File was detonated in
AnyRun VM, receiving a threat score of 100%. Malicious file was quarantined
and then deleted by analyst.

Status: CLOSED | TRUE POSITIVE | DELETED
```

``` title="Triage B"
Description: wscript.exe on LENOVO by marcos
Date: Jun. 18, 2025
Time Quarantined: 13:27:21
Source IP: xx.xxx.xxx.xxx
Destination IP: xx.xxx.xxx.xxx
User: marcos
File Path: \Device\HarddiskVolume3\Windows\System32\wscript.exe
Triggering Indicator: "C:\WINDOWS\System32\WScript.exe" "C:\Users\marcos\Downloads\Pleh.vbs"
Filenames: wscript.exe, Pleh.vbs
SHA-1: e13989a5ba4dba2cbc7c2a779b06f381266c32c7
SHA-256: dc98a3995c8c9db2897b3dcd603d0a55e9d6b42cb3900f9b5666dbb461172197
MD5: 55cde934290e89ae29f92ff118b6280c

Analysis: Malicious script execution detected and prevented on LENOVO host's
user "marcos" with IP address of xx.xxx.xxx.xxx. Script attempted to tamper
with Falcon sensor. VBS script file is detected as a malicious worm with a
score of 47/61 on VirusTotal. File has been quarantined and deleted by analyst.

Status: CLOSED | TRUE POSITIVE | DELETED
```

## On-demand scans

CrowdStrike Falcon supports **on-demand scans** can be run immediately or scheduled for later on selected hosts or host groups. These scans detect and analyze potential security threats on endpoints. On-demand scans are particuarly useful for detecting malicious executables (e.g., .exe, .dll) and do not scan non-executable or archive files by default. This serves as an additional layer of inspection beyond Falcon's real-time protection.

### Scanning

Create a new scan template that will be used by Falcon to scan specific directories within certain endpoints for threats.

1. To create a new scan, expand the sidebar, then navigate to **Endpoint security -> On-demand scans**.

2. In the scan configuration, select when you want the scan to begin, which host machine targets to scan, and the file paths and exclusions.

    !!! info "Scan notifications"

        For Windows endpoints, you will be notified from the endpoint itself of the start/completion of the scan, and if suspicious files were detected.

3. Go back to the Falcon **On-demand scans** page. Click on the most recent scan with the status of **Completed**, then click on **See full details** on the right pane.

4. Here, you can see a list of files that were detected as malicious. Click on the square icon to the right of any detection, and click **Investigate hash**.

5. From here, you can begin triaging like in the previous exercise, using forensic tools like [VirusTotal](https://www.virustotal.com/) and [ANY.RUN](https://app.any.run/) to aid you in your investigation.

## Indicators of Compromise

CrowdStrike Falcon uses **Indicators of Compromise (IOCs)** as digital forensic clues to detect and respond to security breaches. IOCs are pieces of evidence (file hashes, IP addresses, domains, registry keys, or unusual system behaviors) that suggest a device or network has been compromised. Falcon allows security teams to create, manage, and deploy custom IOCs to proactively detect, track, and block known threats in their environment.

One of Falcon's features is **IOC management**. With this, you can add, review, and update indicators like file hashes (SHA256, SHA1, MD5), malicious domains, or IP addresses. You can also specify actions such as *Detect Only* (flag suspicious activity) or *Detect & Prevent* (block immediately). Regular review and auditing of IOCs help maintain effectiveness and reduce false positives. 

Additionally, **Machine Learning (ML) exclusions** in Falcon are used to prevent the ML-based detection engine from flagging certain files, processes, or scenarios that are known to be safe but might otherwise be incorrectly detected as malicious. In SIEM/EDR environments, this configuration of IOCs and ML exclusions is called **tuning**.

### EDR tuning

The first part of this covers the creation of a **custom IOC** for a malicious file. The second part covers the creation of an **ML exclusion** that prevents false positives.

#### Create a custom IOC

Suppose that there is a malicious file that does not get automatically quarantined by the Falcon sensor. One such example from [The-MALWARE-Repo](https://github.com/Da2dalus/The-MALWARE-Repo) is the Trojan executable named **StartBlueScreen.exe**. For this, you will create a custom IOC that detects a malicious file based on its hash value.

1. To create a custom IOC, expand the sidebar and navigate to **Endpoint security -> IOC management**.

2. Now, create an IOC by clicking on the button with the vertical three dots to the right, and click on **Add hashes**:

    * Add the hash value of the malicious file

        !!! note "Add hash"

            Calculate the SHA256 hash of the file by running the following command in the PowerShell terminal:

            ```pwsh
            Get-FileHash -Path "C:\path\to\your\file.ext"
            ```

    * Provide an accurate filename and description

    * Assign the corresponding OS platform tag

    * Label with the appropriate severity level
    
    * Determine the action to take on detection

    The Falcon sensor will now automatically detect the file with this unique hash and quarantine it.

#### Create an ML exclusion

To prevent certain files or folders from being quarantined because they are only used for forensic purposes and don’t pose a threat, create an **ML exclusion**.

1. To create an ML exclusion, expand the sidebar and navigate to **Endpoint security -> Exclusions**.

2. Click on **Create exclusion** in the lower navbar.

3. Select if you want the exclusion to apply to **All hosts** or a **Group of hosts**, and click **Next**.

4. Check that the target(s) is excluded from detections and preventions.

5. Under **Exclusion pattern**, enter the absolute path to the file or directory, or file type that you want to exclude.

6. Add a comment for the audit log, and then click **Create exclusion**.

With this ML exclusion, you can prevent non-threatening files from being quarantined and prevent alerts for false positives.
