# 📁 Remote File Transfer & Retrieval System

## Remote File Transfer & Retrieval System (Client-Server)

**High-Performance File Transfer Solution Based on C Language** | Windows 10/11 | CMake + MinGW

> ⚠️ **Important Notice**: This system is for educational and research purposes only. Please ensure usage in a legal and compliant manner. The client runs silently without UI prompts.

***

## 📚 Table of Contents

- [System Architecture](#architecture)
- [Requirements](#requirements)
- [Build](#build)
- [Configuration](#configuration)
- [Quick Start](#quickstart)
- [FAQ](#faq)
- [Disclaimer](#disclaimer)

***

**🌍 Language / 语言:**
[English](README.en.md) | [简体中文](README.md)

***

## ✨ Core Features

### Client (FileClient.exe) - Background Running, No Window

| Feature | Description |
| :------------ | :-------------------------------- |
| 🔌 **USB Hot-plug** | Automatically detects USB insertion/removal, dynamically adjusts priority |
| ⚡ **Five-Level Priority** | USB Large > USB Medium > Local Large > Local Medium > Small Files |
| 🔄 **Resume Transfer** | Automatic retry on failure (max 5 times) + exponential backoff |
| 🐘 **Large File Support** | Supports 4GB+ file transfers (64-bit operations) |
| 🚀 **Auto Startup** | Automatically registers startup entry on first run |
| ❤️ **Keep-Alive** | TCP KeepAlive to prevent idle disconnections |

**Supported Formats**: PSD | BMP | JPG | JPEG | PNG (customizable extensions)

**File Filter**: Only files larger than 6KB will be transferred

### Server (FileServer.exe) - GUI Window Interface

| Feature | Description |
| :----------- | :-------------- |
| 🧵 **Multi-threaded Concurrency** | Independent thread per client |
| 📊 **Real-time Progress** | Displays transfer progress percentage |
| 📜 **Logging** | GUI display + file persistence |
| 📥 **ACK Confirmation** | Sends confirmation to client upon successful reception |
| 🌐 **Multi-client** | Accepts multiple simultaneous connections |

***

<a name="architecture"></a>

## 🏗️ System Architecture

```
┌─────────────────────┐         TCP (Port 9000)        ┌─────────────────────┐
│      CLIENT         │ ◄═════════════════════════► │      SERVER         │
│                     │                               │                     │
│  ┌───────────────┐  │                               │  ┌───────────────┐  │
│  │ File Scanner  │  │                               │  │ Server Thread │  │
│  │ (Recursive)   │  │                               │  │ (Listen)      │  │
│  └───────┬───────┘  │                               │  └───────┬───────┘  │
│          ▼          │                               │          ▼          │
│  ┌───────────────┐  │                               │  ┌───────────────┐  │
│  │ Task Queue    │  │                               │  │Client Handler │  │
│  │ (5 Priorities)│  │                               │  │(Receive/Save) │  │
│  └───────┬───────┘  │                               │  └───────┬───────┘  │
│          ▼          │                               │          ▼          │
│  ┌───────────────┐  │                               │  ┌───────────────┐  │
│  │Transmission   │  │                               │  │ File I/O      │  │
│  │ Thread        │  │                               │  │ (Write Disk)  │  │
│  └───────────────┘  │                               │  └───────────────┘  │
│                     │                               │     + GUI Log       │
└─────────────────────┘                               └─────────────────────┘
```

**Workflow**:

1. Client starts → Scans USB/D drive/Fixed drives
2. File classification → Builds priority queue
3. Connects to server → Sends metadata (size + filename)
4. Transfers file content in chunks (≤4MB/chunk)
5. Server receives → Writes to disk → Sends ACK confirmation

***

<a name="requirements"></a>

## 📦 Requirements

| Item | Requirement |
| :---------------: | :------------------------------------ |
| **OS** | Windows 10 (1903+) / Windows 11 (64-bit) |
| **CMake** | ≥ 3.20 |
| **MinGW-w64 GCC** | ≥ 12.2.0 |
| **Disk Space** | >2x estimated total file volume |
| **Network** | LAN (client and server can communicate) |

***

<a name="build"></a>

## 🔧 Build

### Method 1: CLion IDE (Recommended)

1. **Open Project**: `File → Open` → Select project root directory
2. **Configure Toolchain**: `Settings → Build → Toolchains` → Select `MinGW`
3. **Build**: `Build → Build Project` (Ctrl+F9)

**Output Locations**:

- Client: `Client\build\Release\FileClient.exe`
- Server: `Server\build\Release\FileServer.exe`

### Method 2: Command Line

```bash
# Build Client
cd Client
mkdir build && cd build
cmake -G "MinGW Makefiles" ..
cmake --build . --config Release

# Build Server
cd ..\..\Server
mkdir build && cd build
cmake -G "MinGW Makefiles" ..
cmake --build . --config Release
```

> ✅ **Note**: The above commands are standard CMake build processes. Ensure CMake (≥3.20) and MinGW-w64 GCC (≥12.2.0) are installed and in your PATH.

***

<a name="configuration"></a>

## ⚙️ Configuration

### Client Configuration (`Client/main.c` lines 16-26)

```c
#define SERVER_IP               "192.168.1.100"  // ⚠️ MUST modify to server IP!
#define SERVER_PORT             9000              // Keep consistent with server
#define BUFFER_SIZE             (4 * 1024 * 1024) // Transfer buffer: 4MB
#define MAX_RETRIES             5                 // Max retry attempts
#define SIZE_30MB               (30 * 1024 * 1024) // Large file threshold: 30MB
```

**Add Supported File Types** (around line 470):

```c
const char* allowed_ext[] = {
    ".psd", ".bmp", ".jpg", ".jpeg", ".png",
    // Add new types here: ".gif", ".pdf", ".mp4", etc.
};
```

### Server Configuration (`Server/server.c` lines 12-18)

```c
#define PORT 9000                      // Listen port (consistent with client)
#define SAVE_DIR "D:\\ReceivedFiles"   // File save directory
#define LOG_FILE "D:\\server_log.txt"  // Log file path
#define BUFFER_SIZE (4 * 1024 * 1024)  // Receive buffer: 4MB
```

***

<a name="quickstart"></a>

## ⚡ Quick Start

### Step 1: Deploy Server

1. **Edit Configuration**: Modify `SAVE_DIR` and `LOG_FILE` paths in `Server/server.c`
2. **Compile**: See [Build](#build) section
3. **Run**: Double-click `FileServer.exe`
4. **Verify**: Window displays `[INFO] Server is listening on port 9000`

### Step 2: Deploy Client

1. **Edit Configuration**: Modify `SERVER_IP` in `Client/main.c` to the actual server IP
2. **Compile**: See [Build](#build) section
3. **Run**: Double-click `FileClient.exe` (no window, runs in background)
4. **Verify**: Command line displays `[INFO] Found X eligible files`

### Step 3: Test Transfer

1. **Prepare Test Files**: Place `.jpg/.png` files (>6KB) on USB or D drive
2. **Observe Server Logs**: Should display `[PROGRESS] xxx - XX%` and `[SUCCESS] Received xxx`
3. **Check Receive Directory**: Files should appear in `SAVE_DIR`

***

<a name="faq"></a>

## ❓ FAQ

### Q1: Cannot connect to server?

**Checklist**:

- ✅ Is the server running?
- ✅ Is port 9000 listening? (`netstat -an | findstr :9000`)
- ✅ Is the IP address correct? (ping test)
- ✅ Is the firewall allowing port 9000?

**Quick Fix**:

```bash
# Add firewall rule (administrator privileges)
netsh advfirewall firewall add rule name="FileTransfer" dir=in action=allow protocol=TCP localport=9000
```

### Q2: Transfer failed?

**Common Causes and Solutions**:

| Cause | Solution |
| :----: | :---------------------------------- |
| Network interruption | Check cable/WiFi, wait for auto-retry (max 5 times) |
| Insufficient disk space | Clean server disk or change SAVE_DIR |
| Antivirus blocking | Add FileClient.exe and FileServer.exe to whitelist |
| File in use | Close the program using the file |
| Buffer overflow | Reduce BUFFER_SIZE to 2MB |

### Q3: No response to USB insertion?

**Diagnostic Steps**:

1. Run client as administrator
2. Insert USB and observe command line output for `[INFO] USB device inserted`
3. Ensure USB format is NTFS/FAT32/exFAT
4. If in a virtual machine, manually pass through the USB device

### Q4: How to disable auto startup?

**Method 1**: Delete registry entry

```
Win+R → regedit → Navigate to:
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
Delete "Zafkiel" value
```

**Method 2**: Command line (Windows CMD or PowerShell)

```cmd
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v Zafkiel /f
```

> ⚠️ **Note**:
>
> - In Windows CMD/PowerShell, use backslash `\` for paths, no additional escaping needed
> - `/f` parameter forces deletion without confirmation
> - This command requires administrator privileges

### Q5: How to change port?

Modify both locations and recompile:

```c
// Client/main.c
#define SERVER_PORT 9001

// Server/server.c
#define PORT 9001
```

***

<a name="disclaimer"></a>

## 📄 Disclaimer

> ⚖️ **Legal Notice and Usage Terms - Please Read Carefully**

### 1. Copyright and License Statement

**© 2026 Remote File Transfer System. All rights reserved.**

The intellectual property rights of this software (hereinafter referred to as "this system") and its related documentation and source code belong to the author. This system is released in open source form, following these principles:

1. **Provided As-Is**: This software is provided "AS-IS" without any express or implied warranties, including but not limited to warranties of merchantability, fitness for a particular purpose, and non-infringement.
2. **Usage Risk**: Users assume all risks of using this system. The author is not responsible for any losses caused by using or inability to use this system.
3. **Open Source Spirit**: This system follows the open source sharing concept. Learning and exchange are welcome, but please respect the author's work and intellectual property rights.

### 2. Usage Restrictions and Prohibited Activities

**🚫 Strictly Prohibited Behaviors**:

| Prohibited Category | Detailed Description | Possible Legal Consequences |
| :--------: | :------------------------------------- | :-------------------------------------------------- |
| **Illegal Data Collection** | Collecting or transferring others' private files, sensitive data, or personal information without explicit consent | Violation of Article 285 of the Criminal Law of the People's Republic of China (Illegal Acquisition of Computer Information System Data Crime), may face 3-7 years imprisonment |
| **Commercial Espionage** | Using this system to steal competitors' trade secrets, technical documents, customer data, etc. | Violation of the Anti-Unfair Competition Law, may bear civil compensation liability and criminal liability |
| **Network Attack Tool** | Using this system's technical features for unauthorized access, data theft, malware distribution, etc. | Violation of Article 27 of the Cybersecurity Law, may face fines and criminal liability |
| **Privacy Violation** | Automatically collecting and transferring others' personal files, photos, contacts, etc. without their knowledge | Violation of Articles 1032-1034 of the Civil Code (Privacy Protection), must bear civil compensation liability |
| **Malicious Content Distribution** | Using this system to spread viruses, Trojans, phishing files, illegal information, etc. | Violation of Articles 46-48 of the Cybersecurity Law, may face administrative detention and criminal liability |

**📋 Compliant Usage Requirements**:

1. **Enterprise Internal Use**: If deploying this system in an enterprise environment for legitimate business needs (such as file backup, data synchronization), you must:
   - Establish clear data management policies and obtain IT department approval
   - Inform all relevant personnel of the system's existence and working principles
   - Ensure no violation of employee privacy rights and company information security policies
   - Regularly audit transfer records to prevent abuse
2. **Educational and Research Use**: If using this system for teaching demonstrations, technical research, or academic experiments, you must:
   - Clearly indicate it is for educational purposes only
   - Do not test in real production environments without authorization
   - Respect data ownership and privacy rights
   - Cite the source when publishing research results
3. **Personal Learning Use**: When using this system to learn C language programming, network communication, system development, etc., you should:
   - Test only on your own devices and with your own data
   - Do not operate on others' devices or data
   - Comply with laws, regulations, and ethical standards

### 3. Data Security and Technical Limitations

**⚠️ Current Version Technical Limitations**:

| Security Risk | Risk Level | Detailed Description | Recommended Measures |
| :--------: | :---: | :---------------------------- | :-------------------------------- |
| **No Encryption** | 🔴 High | All files are transmitted in plaintext over the network, may be intercepted by man-in-the-middle attacks | Recommended for internal network/VPN use only; future versions plan to add TLS/SSL encryption support |
| **No Authentication** | 🔴 High | Any device knowing the IP and port can connect to the server, lacks identity verification mechanism | Configure firewall rules to restrict access sources; recommend adding Token or certificate authentication |
| **Plaintext Storage** | 🟡 Medium | IP addresses, paths, etc. in configuration files are stored in plaintext | Delete configuration copies immediately after use; avoid committing to public code repositories |
| **No Access Control** | 🟡 Medium | Lacks user permission management, operation audit, etc. | Use in trusted networks; regularly review log files |
| **Single Point of Failure** | 🟢 Low | Server crash will cause all clients to be unable to transfer | Recommend regularly restarting server; monitor process status |

**🔒 Sensitive Data Processing Warning**:

**Absolutely Prohibited Data Types** (not recommended even with authorization):

- 🔴 Bank account information, credit card numbers, payment passwords
- 🔴 ID document photos (ID card, passport, driver's license)
- 🔴 Medical health records, medical history files
- 🔴 Company core secrets, business contracts, financial statements
- 🔴 Personal private photos, videos, chat records
- 🔴 Passwords, private keys, digital certificates, encryption seeds

**If Sensitive Data Must Be Transferred** (with explicit written authorization):

1. Use an independent encrypted channel (such as VPN, SSH tunnel) to wrap the TCP connection
2. Encrypt files with AES-256 or other strong encryption before transfer
3. Immediately delete temporary files from both ends after transfer
4. Keep complete operation logs for review
5. Regularly change transfer credentials and encryption keys

### 4. Network Security Usage Guidelines

**✅ Recommended Security Practices**:

1. **Network Environment Isolation**
   - 🟢 **Best Practice**: Deploy in a completely isolated internal network environment (physical isolation or VLAN isolation)
   - 🟡 **Acceptable**: Connect to remote server through VPN tunnel
   - 🟡 **Minimum Requirement**: Use in trusted home/office WiFi or wired network
   - 🔴 **Avoid**: Use in public WiFi (coffee shops, airports, hotels), insecure open networks
2. **Firewall Configuration Example**

```bash
# Windows Firewall Advanced Configuration (Recommended)
netsh advfirewall firewall add rule name="FileTransfer-In" ^
    dir=in action=allow protocol=TCP localport=9000 ^
    remoteip=192.168.1.0/24 enable=yes profile=private

# Explanation:
# - Only allows connections from 192.168.1.0/24 subnet (internal network IP range)
# - Only effective under Private Profile
# - Records all inbound connection attempts for audit
```

3. **Regular Security Checklist**
   - [ ] Review transfer logs weekly for abnormal connections
   - [ ] Update OS patches and security software monthly
   - [ ] Rotate server listening port quarterly
   - [ ] Evaluate whether there are more secure alternatives semi-annually
   - [ ] Follow this project's GitHub Releases page for security updates
4. **Abnormal Situation Response Strategy**
   - **Unknown connection detected**: Immediately disconnect network, check if system is compromised
   - **File loss or corruption**: Restore from backup, investigate if malicious tampering occurred
   - **Sudden performance degradation**: Check if suffering DoS attack or resource exhaustion attack
   - **Legal complaint received**: Immediately stop using, preserve log evidence, consult professional lawyer

### 5. Legal Liability and Dispute Resolution

1. **Liability Limitation Clause**
   - The author is only responsible for the **functional implementation and technical correctness** of this system, and is not responsible for the following:
     - Legal liability caused by user misuse
     - Third parties using this system for illegal activities
     - Economic losses caused by data leakage, loss, or damage
     - Secondary disasters caused by system vulnerabilities being exploited
     - Service unavailability due to network interruption, hardware failure, etc.
   - **Maximum Compensation Limit**: If the court orders the author to bear compensation liability, the compensation amount shall not exceed the fee paid by the user for this system (this system is free, so the compensation amount is 0)
2. **Applicable Law and Jurisdiction**
   - The formation, validity, interpretation, performance, and dispute resolution of this statement shall be governed by the **Laws of the People's Republic of China**
   - Any dispute arising from or related to this statement shall first be resolved through friendly consultation
   - If consultation fails, either party has the right to file a lawsuit with the **People's Court with jurisdiction at the author's location**
   - Costs incurred during litigation (including but not limited to litigation fees, attorney fees, appraisal fees, travel expenses, etc.) shall be borne by the losing party
3. **Integrity Clause**
   - This statement constitutes the complete agreement between both parties regarding the use of this system, superseding all previous oral or written agreements
   - If any provision of this statement is deemed invalid or unenforceable, it shall not affect the validity of other provisions
   - The author's failure to exercise or delay in exercising any right under this statement shall not constitute a waiver of that right
4. **Updates and Notices**
   - The author reserves the right to modify this statement at any time. The modified statement will be published in the GitHub repository in the form of a new Commit
   - Continued use of the updated version constitutes acceptance of the modified statement terms
   - Major changes will be notified 30 days in advance through GitHub Release Notes, Issue announcements, etc.
5. **Final Interpretation Right**
   - The final interpretation right of this statement belongs to the author
   - If there is a conflict between the Chinese version and other language versions, the Chinese version shall prevail
   - For any questions about this statement, please contact the author through GitHub Issues

### 6. Special Statement

**🎓 Academic Integrity Statement**:
This system can be used as a teaching case for computer science, software engineering, network security, and other courses, but students shall not directly copy the source code as their own homework or graduation project submission. If reference is needed, please indicate the source and understand the implementation principles before completing independently.

**♿ Accessibility Statement**:
The current version of this system is a command line/GUI interface and does not currently support screen readers and other assistive technologies. Future versions will consider adding accessibility support.

**🌍 Internationalization Statement**:
This system currently only supports Simplified Chinese interface and documentation. Community contributions for multi-language support (English, Japanese, Korean, etc.) are welcome.

**🔄 Forward Compatibility Statement**:
The author commits to maintaining backward compatibility of configuration file formats, API interfaces, and communication protocols in future versions. If breaking changes are necessary, they will be clearly stated in Release Notes and migration guides will be provided.

***

**Document Last Updated**: May 1, 2026\
**Contact**: Please submit issues and feedback through GitHub Issues
