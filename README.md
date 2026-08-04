<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/local-file-inclusion/main/content/local-file-inclusion.svg"></p>

## Local File Inclusion (LFI)
Local File Inclusion (LFI) is a web application vulnerability that occurs when an application includes a file from the local file system based on untrusted user input. If an attacker can manipulate the file path used by a file inclusion function, they may gain access to sensitive files, disclose application source code, or, under certain circumstances, execute arbitrary code by tricking the application into including a malicious local file.

## How LFI Works
1. Identify Vulnerable Parameters: Attackers look for parameters in web applications that are used to include files, such as `include()`, `require()`, `include_once()`, or `require_once()` in PHP.
2. Craft Malicious Input: Attackers create input that includes a path to a local file they wish to access.
3. Exploit the Vulnerability: The attacker sends the malicious input to a vulnerable parameter, prompting the web application to read the specified local file.

## Impact of LFI
- Data Theft: Sensitive information, including databases, configuration files, and credentials, may be accessed or stolen.
- Server Compromise: Attackers may gain full control of the vulnerable server.
- Privilege Escalation: If the web server has excessive privileges, attackers may obtain broader access to the operating system.
- Reputation and Financial Damage: Organizations may face service disruptions, financial losses, regulatory penalties, and damage to customer trust.

## LFI Mitigation Strategies
- Do Not Trust User Input for File Inclusion: Avoid allowing users to directly specify file names or paths. Instead, use predefined mappings or allowlists.
- Disable Local File Inclusion: Only enable the local file inclusion feature when absolutely necessary.
- Validate User Input: If user input must be accepted, ensure it is validated against a predefined list of acceptable values. Do not allow arbitrary file names or URLs.
- Follow the Principle of Least Privilege: Configure the web server with only the permissions necessary for its intended functionality. This practice limits potential damage if an attacker successfully exploits the application.
- Keep Software Updated: Regularly apply security updates to reduce exposure to known vulnerabilities.
- Use a Web Application Firewall (WAF): A WAF can help detect and block malicious requests aimed at exploiting file inclusion vulnerabilities.
- Perform Regular Security Testing: Conduct routine security assessments to identify insecure file inclusion before attackers can exploit it.

## LFI Example
Clone this current repo recursively
```sh
git clone --recurse-submodules https://github.com/qeeqbox/local-file-inclusion
```
Run the webapp using Python
```sh
python3 local-file-inclusion/vulnerable-web-app/webapp.py
```
Open the webapp in your browser 127.0.0.1:5142
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/local-file-inclusion/main/content/1.png"></p>
Use the admin default credentials (username: admin and password: admin) to log in 
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/local-file-inclusion/main/content/2.png"></p>
Click Open Full Log to review the entire logs file
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/local-file-inclusion/main/content/3.png"></p>
The logs file locaiton is logs/httpd.log and is reviewed as raw file
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/local-file-inclusion/main/content/4.png"></p>
Change the logs file to any internal file, such as ../webapp.py, and press Enter. The file will be retrieved, and the webapp.py source code will be shown.
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/local-file-inclusion/main/content/5.png"></p>

## Code
When a user requests the entire logs file to review, the logs location can be edited

```py
def do_GET(self):
    ...
    ...
    elif parsed_url.path == "/logs":
        if parsed_url.query.startswith("file=") and "file" in get_request_data:
            self.send_content(200, [('Content-type', 'text/html')], self.read_logs(get_request_data["file"][0]))
            return
    ...
    ...
```
The read_logs() function does not verify which file to read from, allowing any internal file to be accessed
```py
def read_logs(self, file, search=None, recent_rows=10):
    temp_logs = b""
    if search:
        with open(path.join(LOGS_FOLDER,file),"r") as f:
            lines = f.readlines()
            found = [line for line in lines if search in line]
            if found:
                temp_logs += f"<div>Number of lines: {len(found)}</div>".encode("utf-8")
                for line in found[-recent_rows:]:
                    temp_logs += f"<div>{line.strip()}</div>".encode("utf-8")
            if temp_logs == b"":
                temp_logs = f"<div>No match found: {search}</div>".encode("utf-8")
    else:
        with open(path.join(LOGS_FOLDER,file),"rb") as f:
            temp_logs += f.read()
    return temp_logs
```
