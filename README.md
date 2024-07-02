# LFI-RFI
## Table of contents
1. [File inclusion](#file-inclusion)
2. [What are the differences between File Inclusion vs. Directory Path Traversal?](#what-are-the-differences-between-file-inclusion-vs-directory-path-traversal)
3. [Types of File Inclusion Attacks](#types-of-file-inclusion-attacks)
4. [How to Prevent LFI and RFI Attacks?](#how-to-prevent-lfi-and-rfi-attacks)
## File Inclusion
A file inclusion vulnerability enables an attacker to gain unauthorized access to sensitive files on a web server or execute malicious files through the utilization of the ‘include’ functionality. These attacks are typically associated with web applications and can have serious security effects if not properly mitigated.

The consequences of these vulnerabilities can range from data exposure and server compromise to the defacement of websites and unauthorized code execution.

## What are the differences between File Inclusion vs. Directory Path Traversal?
Directory Path Traversal can often be a means to exploit File Inclusion vulnerabilities. If an attacker can manipulate the file path, they can use it to achieve LFI (Local File Inclusion).

While Directory Path Traversal primarily focuses on manipulating the file system path, File Inclusion vulnerabilities deal with including external files (either locally or remotely).
## Types of File Inclusion Attacks
- Local File Inclusion (LFI):

  * In an LFI attack, an attacker exploits a vulnerability in a web application to include local files stored on the server.

  * Typically, the attacker manipulates user-controllable input (e.g., URL parameters or cookies) to specify the file path to include.

- Remote File Inclusion (RFI):
  
  * In an RFI attack, an attacker exploits a vulnerability to include files from a remote server or location, usually using user-controllable input.

  * RFI attacks can lead to remote code execution, allowing the attacker to run arbitrary code hosted on a remote server.

  * RFI attacks are more dangerous than LFIs because they enable attackers to execute malicious code on the target server, potentially compromising security.

## How to Prevent LFI and RFI Attacks?
- Input Validation
- Whitelist Approach
- Use Absolute Paths (LFI)
- Use Local Copies (RFI)
- Access Controls
- Avoid User-Controllable Inclusion
- Security Headers (RFI)
- Network Segmentation (RFI)
- Regular Updates and Patching (RFI)
- Use a WAF (Web Application Firewall)
