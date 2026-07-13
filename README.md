<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/local-file-inclusion/main/content/local-file-inclusion.svg"></p>

An application retrieves local files without properly validating or restricting the provided file path. A threat actor can exploit this vulnerability by manipulating the file path to gain access to sensitive or unintended files on the underlying system. By doing so, the attacker could cause the application to read internal files that contain threat actor information. This may lead to unauthorized disclosure of information and data theft.

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
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/remote-file-inclusion/main/content/4.png"></p>
Change the logs file to any internal file, such as ../webapp.py, and press Enter. The file will be retrieved, and the webapp.py source code will be shown.
<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/remote-file-inclusion/main/content/5.png"></p>

## Code
When a user requests the entire logs file to review, the logs location can be editted

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