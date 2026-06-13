---
title: "Building Port Scanner in Python"
date: 26-06-02 +1030
categories: [python, Networking, reconnaissance]
tags: [cybersecurity, python, github, networking, beginner]
---

## Building a Network Port Scanner with Python: From Single Port to Multi-Threaded Automation

If you are stepping into the world of cybersecurity or network engineering, you will quickly realize that visibility is everything. You cannot secure what you do not know exists. One of the foundational skills every security professional must master is understanding how devices communicate across a network and how to audit those communication channels.

In this guide, we are going to build a network port scanner from scratch using Python. We will start with a basic script that checks a single port and step-by-step evolve it into a high-performance, multi-threaded, data-exporting security tool.

### Introduction: The Basics of Ports and Scanning

Before diving into the programming part, let’s learn the foundational concepts of networking and security that make port scanning necessary.

#### What is a Port and Why Do They Matter?

Think of an IP address as the street address of an apartment building. If you want to mail a letter to someone in that building, the street address gets you to the front door, but you still need the specific apartment number to reach the right person.

In networking, an IP address identifies a physical device on a network, while a port is a virtual number that routes traffic to a specific service or application running on that device. Ports are numbered from 0 to 65535. For example:

| Port | Service |
| ---- | ------- |
| 22   | SSH     |
| 80   | HTTP    |
| 443  | HTTPS   |
| 3306 | MySQL   |

#### Why Port Scanning is Crucial in Cybersecurity

A port scanner is a tool that sends requests to specific ports on a target device to see which ones are listening (open) and which ones are closed or blocked by a firewall.

In cybersecurity, port scanning is used for:

* Asset Inventory: Identifying what services are running across an organization’s network.
* Vulnerability Assessment: Finding outdated, misconfigured, or unauthorized services that malicious actors could exploit.
* Firewall Verification: Ensuring that your network security rules are actively blocking public access to internal services.

#### Security Concerns and Ethics

⚠️ Important Legal Warning: Port scanning can be interpreted as a precursor to a cyberattack. Never scan an IP address, domain, or network that you do not explicitly own or have written authorization to test. If you want to practice safely, use your local loopback address (127.0.0.1) or set up private virtual machines in a home lab environment.

#### Why Python is the Ultimate Cybersecurity Language

Python is considered as the Swiss Army knife of the cybersecurity industry. While heavy-duty tools like nmap are excellent for production scanning, knowing how to write your own tools in Python gives you a competitive edge because:

* Rapid Prototyping: You can write functioning network tools in just a few lines of clean, readable code.
* Automation: Python easily ties into other security tools, databases, and APIs.
* No Third-Party Dependencies: Python’s standard library comes pre-equipped with powerful modules (like socket and json) capable of handling low-level network communications right out of the box.

### Setting Up Your Lab Environment

To run the scripts in this tutorial, we need to use our computer's terminal interface.

**For Windows Users (Command Prompt / PowerShell)**

Open the Start menu, type cmd or powershell, and press Enter. Verify Python is installed by typing:

```bash
python --version
```

We can run our scripts by navigating to the folder where we saved our file (e.g., cd Documents) and typing:

```bash
python scanner.py 127.0.0.1 80
```

**For Linux & macOS Users (Terminal)**

Open your Terminal application (Ctrl+Alt+T on most Linux distributions).

Check your Python installation (most Linux/macOS systems use python3 explicitly):

```bash
python3 --version
```

Run the script by typing:

```bash
python3 scanner.py 127.0.0.1 80
```

### **1. Building a Simple Port Scanner in Python (Version 1)**

In this first version of our port scanner, we will create a simple TCP port scanner that:

* Accepts a target IP address from the command line
* Accepts multiple port numbers to scan
* Attempts to connect to each port
* Reports whether each ports are open or closed

This project introduces the basics of Python sockets and TCP port scanning. You can access the full code from here: <https://github.com/Ramiyakaru/python-port-scanner/blob/main/v1-basic-port-scanner/scanner_v1.py>

#### 1. Importing Required Modules

```python
import socket
import sys
```

Python provides built-in modules that extend its functionality.

#### The `socket` Module

```python
import socket
```

The `socket` module allows applications to communicate over a network. A socket acts as an endpoint that can send or receive data between devices. In this project, sockets are used to attempt TCP connections to remote ports.

#### The `sys` Module

```python
import sys
```

The `sys` module gives access to command-line arguments supplied when running the script.
For example:

```bash
python scanner.py 192.168.1.10 80 443 22
```

In this command:

* `192.168.1.10` is the target IP address
* `80`, `443`, and `22` are the ports to scan

##### 2. Creating the Port Scanning Function

```python
def scan_port(host, port):
```

This function performs the actual port scan.

It accepts two parameters:

* `host` – The target IP address
* `port` – The port number to test

The function returns:

* `True` if the port is open
* `False` if the port is closed or unreachable

Encapsulating the scanning logic inside a function makes the code easier to reuse and maintain.

#### 3. Creating a TCP Socket

```python
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```

This line creates a TCP socket.

### Understanding `AF_INET`

```python
socket.AF_INET
```

`AF_INET` specifies that the socket will use IPv4 addresses.

Examples of IPv4 addresses include:

```text
192.168.1.1
10.0.0.5
8.8.8.8
```

### Understanding `SOCK_STREAM`

```python
socket.SOCK_STREAM
```

`SOCK_STREAM` specifies that the socket will use the TCP protocol.

TCP provides:

* Reliable communication
* Error checking
* Connection-oriented communication

Because many network services use TCP, it is ideal for a basic port scanner.

#### 4. Setting a Timeout

```python
sock.settimeout(1)
```

This sets a timeout of one second for connection attempts. Without a timeout, the scanner could wait indefinitely for a response from an unresponsive host.

Benefits of using a timeout include:

* Faster scans
* Better performance
* Improved user experience

#### 5. Attempting a Connection

```python
result = sock.connect_ex((host, port))
```

This is the core operation of the port scanner. The scanner attempts to establish a TCP connection to the specified host and port.

For example:

```python
("192.168.1.10", 80)
```

### Why Use `connect_ex()`?

Unlike `connect()`, which raises an exception when a connection fails, `connect_ex()` returns a status code.

Common return values:

| Return Value | Meaning               |
| ------------ | --------------------- |
| 0            | Connection successful |
| Non-zero     | Connection failed     |

This makes it easy to determine whether a port is open.

#### 6. Determining Whether a Port Is Open

```python
if result == 0:
    return True
else:
    return False
```

If the connection attempt succeeds, `connect_ex()` returns `0`. A successful connection means a service is listening on that port.

Example:

```text
[+] Port 80 OPEN
```

If the return value is anything other than `0`, the connection failed.

Example:

```text
[-] Port 21 CLOSED
```

#### 7. Handling Errors

```python
except Exception:
    return False
```

Network operations can fail for many reasons:

* Invalid IP addresses
* Network connectivity problems
* Firewall restrictions
* DNS resolution failures

Instead of crashing, the scanner catches the error and safely returns `False`. This improves reliability and keeps the program running.

#### 8. Closing the Socket

```python
finally:
    sock.close()
```

The `finally` block always executes, regardless of whether an exception occurs.

Closing the socket ensures that system resources are released properly. This is an important networking best practice because it prevents resource leaks and excessive open connections.

#### 9. Program Entry Point

```python
if __name__ == "__main__":
```

This is a common Python convention. It ensures that the code only executes when the script is run directly.

For example:

```bash
python scanner.py
```

Everything inside this block serves as the main program logic.

#### 10. Validating User Input

```python
if len(sys.argv) < 3:
```

The scanner expects at least:

1. Script name
2. Target IP address
3. One port number

If fewer arguments are provided, the program displays usage instructions.

```python
print("Usage: python scanner.py <Target IP> <Port1> <Port2> ...")
```

Example output:

```text
Usage: python scanner.py <Target IP> <Port1> <Port2> ...
Example: python scanner.py 192.168.1.10 22 80 443
```

The script then exits using:

```python
sys.exit(1)
```

#### 11. Extracting the Target Host

```python
target_host = sys.argv[1]
```

The first user-supplied argument becomes the target host.

For example:

```bash
python scanner.py 192.168.1.10 22 80 443
```

Results in:

```python
target_host = "192.168.1.10"
```

#### 12. Extracting Port Numbers

An empty list is created to store valid port numbers.

```python
ports_to_scan = []
```

The scanner then loops through every argument after the target IP address.

```python
for arg in sys.argv[2:]:
```

For example:

```bash
python scanner.py 192.168.1.10 22 80 443
```

Produces:

```python
22
80
443
```

Each port is converted into an integer.

```python
ports_to_scan.append(int(arg))
```

### Handling Invalid Input

If the user enters something that is not a valid number:

```bash
python scanner.py 192.168.1.10 abc 80
```

Python raises a `ValueError`.

The scanner catches the error and ignores the invalid port.

```python
except ValueError:
    print(f"[-] Ignoring invalid port: {arg}")
```

Output:

```text
[-] Ignoring invalid port: abc
```

#### 13. Displaying Scan Information

```python
print(f"\nStarting scan on host: {target_host}")
print("-" * 40)
```

Example output:

```text
Starting scan on host: 192.168.1.10
----------------------------------------
```

This provides a clean header before scanning begins.

#### 14. Scanning Each Port

```python
for target_port in ports_to_scan:
```

The scanner loops through every port supplied by the user.

For each port, the `scan_port()` function is called.

```python
is_open = scan_port(target_host, target_port)
```

The function returns either:

* `True` (open)
* `False` (closed or filtered)

#### 15. Displaying Results

If the port is open:

```python
print(f"[+] Port {target_port} OPEN")
```

Example output:

```text
[+] Port 80 OPEN
```

Otherwise:

```python
print(f"[-] Port {target_port}: CLOSED or FILTERED")
```

Example output:

```text
[-] Port 23: CLOSED or FILTERED
```

### Why "Closed or Filtered"?

The scanner cannot always determine whether:

* The port is truly closed
* A firewall silently blocked the connection

Because of this limitation, the result is displayed as:

```text
CLOSED or FILTERED
```

#### 16. Finishing the Scan

```python
print("-" * 40)
print("Scan complete")
```

Example output:

```text
----------------------------------------
Scan complete
```

This indicates that all requested ports have been tested. This is a sample output of the scan.

![Terminal output showing the Version 1 port scanner identifying open and closed ports on a target host.](/assets/img/blog10/1-v1-output.webp)
*Figure 1: Output from Version 1 of the Python port scanner displaying the status of user-specified ports.*

---

### **2. Building a Port Scanner in Python (Version 2) – Adding Port Range Support**

#### Introduction

In Version 1, we built a simple TCP port scanner that could test individual ports supplied through the command line.

For example:

```bash
python scanner.py 192.168.1.10 22 80 443
```

While this works well for a small number of ports, it becomes impractical when scanning larger groups of ports. Imagine trying to scan the first 1,000 ports manually:

```bash
python scanner.py 192.168.1.10 1 2 3 4 5 6 7 8 ...
```

Clearly, there must be a better way.
In Version 2, we introduce **port range support**, allowing us to specify ranges such as:

```bash
python scanner.py 192.168.1.10 1-1000
```

The scanner will automatically expand the range and scan every port within it. This enhancement makes the scanner significantly more practical for real-world use.

#### What's New in Version 2?

Compared to Version 1, we are adding:

* Support for port ranges
* A dedicated port parsing function
* More flexible command-line input
* Improved code organization
* Cleaner code
* Real-time progress tracking with the tqdm library

#### The UX Problem: Silent Execution vs. Real-Time Feedback

Before we write the new code, we have to think about User Experience (UX). When we scale up our scanner to check thousands of ports concurrently, network operations take time. If we run a script and the terminal remains completely blank for 15 seconds, we will think: Is it working? Did it freeze? Should I cancel it?

To solve this, we are integrating a popular third-party Python library called **tqdm**. It automatically calculates execution speed and displays a dynamic visual progress bar right in our terminal.

Because tqdm is a third-party package, we will need to install it in our environment before running Version 3. Open the terminal or command prompt and run:

```bash
pip install tqdm
```

#### 1. Importing new modules

```python
from tqdm import tqdm
```

We import the tqdm class, which wraps around any Python iterable (or iterator) to track how many items have been processed out of the total payload.

#### Why Add Port Range Support?

In Version 1, users had to enter every port manually.
For example:

```bash
python scanner.py 192.168.1.10 22 80 443
```

This works well for a few ports but becomes tedious when scanning hundreds or thousands of ports. Version 2 allows users to specify ranges such as:

```bash
python scanner.py 192.168.1.10 1-1000
```

The scanner automatically expands the range and scans all ports between 1 and 1000.

#### New Feature: The `parse_ports()` Function

The biggest addition in Version 2 is the `parse_ports()` function.

```python
def parse_ports(port_args):
```

Its responsibility is to process the user's input and convert it into a list of valid port numbers. For example:

Input:

```bash
python scanner.py 192.168.1.10 80
```

Produces:

```python
[80]
```

Input:

```bash
python scanner.py 192.168.1.10 20-25
```

Produces:

```python
[20, 21, 22, 23, 24, 25]
```

This function separates input processing from scanning logic, making the code easier to maintain.

#### 2. Creating the Port List

The function starts by creating an empty list.

```python
ports = []
```

This list will store every valid port discovered during processing.

#### 3. Processing User Input

The function loops through every argument supplied by the user.

```python
for arg in port_args:
```

For example:

```bash
python scanner.py 192.168.1.10 22 80 100-105
```

The loop processes:

```text
22
80
100-105
```

One item at a time.

#### 4. Detecting Port Ranges

To determine whether the user supplied a range, the scanner checks for a hyphen.

```python
if "-" in arg:
```

Examples of valid ranges:

```text
1-100
20-25
8000-8100
```

If a hyphen exists, the scanner treats the input as a range rather than a single port.

#### 5. Splitting the Range

Once a range is detected, it must be separated into a starting port and ending port.

```python
start, end = arg.split("-")
```

Example:

```python
"20-25"
```

Becomes:

```python
start = "20"
end = "25"
```

The values are then converted into integers.

```python
start = int(start)
end = int(end)
```

Result:

```python
start = 20
end = 25
```

#### 6. Generating Every Port in the Range

The scanner uses Python's built-in `range()` function.

```python
for port in range(start, end + 1):
```

The `+1` is important because Python excludes the ending value when generating ranges.

Without `+1`:

```python
range(20, 25)
```

Produces:

```text
20
21
22
23
24
```

With `+1`:

```python
range(20, 25 + 1)
```

Produces:

```text
20
21
22
23
24
25
```

Each port is then added to the list.

```python
ports.append(port)
```

#### 7. Handling Invalid Ranges

Users may accidentally provide invalid ranges.

Example:

```bash
python scanner.py 192.168.1.10 abc-def
```

Attempting to convert these values into integers causes a `ValueError`. To prevent the program from crashing, the scanner uses exception handling.

```python
except ValueError:
    print(f"[-] Invalid range ignored: {arg}")
```

Output:

```text
[-] Invalid range ignored: abc-def
```

The scanner simply skips the invalid range and continues running.

#### 8. Supporting Individual Ports

Not every argument will be a range. If no hyphen exists, the scanner treats the value as a single port.

```python
else:
```

Example:

```bash
80
443
8080
```

Each value is converted into an integer.

```python
ports.append(int(arg))
```

Result:

```python
[80, 443, 8080]
```

#### 9. Why Create a Separate Function?

In Version 1, port processing occurred directly within the main program.
As software projects grow, placing too much logic inside the main program makes maintenance and debugging difficult.
By moving the logic into a dedicated function, we gain several benefits.

#### 10. Implementing the progress bar

we looped over our ports using a standard for target_port in ports_to_scan: statement. In Version 2, we wrap that list directly inside tqdm():

```python
for port in tqdm(
    ports_to_scan,
    desc="Scanning Ports",
    unit="port",
    ncols=100
):
```

* **ports_to_scan**: The target list of integers we want to loop through. tqdm measures the total length of this list to calculate the exact percentage (%) completed in real time.

* **desc="Scanning Ports"**: This adds a clean text prefix to the left side of our progress bar, clearly stating what the script is doing.

* **unit="port"**: Changes the default measurement label from iterations-per-second (it/s) to ports-per-second (port/s), making our security tool look and feel completely custom.

* **ncols=100**: Locks the total width of the progress bar to 100 characters. This ensures the UI stays neat and doesn't break or wrap awkwardly on smaller terminal windows.

#### Better Readability

The main program becomes easier to understand.

#### Better Reusability

The function can be reused elsewhere in the project.

#### Easier Maintenance

Future enhancements can be made without modifying the scanning logic. This design principle is commonly known as **separation of concerns**.

#### Updated Usage Examples

Version 2 supports several input formats.

#### **Scan a Single Port**

```bash
python scanner.py 192.168.1.10 80
```

#### **Scan Multiple Ports**

```bash
python scanner.py 192.168.1.10 22 80 443
```

#### **Scan a Port Range**

```bash
python scanner.py 192.168.1.10 1-1000
```

#### **Mix Individual Ports and Ranges**

```bash
python scanner.py 192.168.1.10 22 80 443 8000-8100
```

This flexibility makes the scanner much more useful in real-world scenarios. This is a sample output of the scan.

![Terminal output demonstrating Version 2 port range scanning functionality.](/assets/img/blog10/2-v2-output.webp)
*Figure 2: Version 2 scanner successfully processing a port range and displaying the scan results.*

---

### **3. Building a Port Scanner in Python (Version 3) – Speeding Up Scans with Multithreading**

#### Introduction

In Version 2, we improved our scanner by adding support for port ranges. This allowed us to scan hundreds or even thousands of ports using commands such as:

```bash
python scanner.py 192.168.1.10 1-1000
```

While this was a significant improvement, a new problem quickly becomes noticeble. Scanning ports one at a time can be very slow. Since each connection attempt waits up to one second for a response, scanning hundreds of ports may take several minutes to complete.

To solve this problem, Version 3 introduces **multithreading**, allowing multiple ports to be scanned simultaneously. This dramatically improves scanning speed and moves our project closer to the behavior of professional port scanning tools.

#### What's New in Version 3?

Compared to Version 2, we added:

* Multithreaded scanning
* The `ThreadPoolExecutor` class
* A worker function for concurrent tasks
* Faster scanning of large port ranges

As a result, the scanner can test many ports at the same time instead of waiting for each scan to finish before starting the next one.

#### The Problem with Sequential Scanning

In Versions 1 and 2, ports were scanned one after another.

For example:

```text
Scan Port 22
Wait for result

Scan Port 80
Wait for result

Scan Port 443
Wait for result

Scan Port 8080
Wait for result
```

This approach is called **sequential processing**. While simple, it becomes inefficient when scanning large numbers of ports. Imagine scanning 1,000 ports with a one-second timeout.

In the worst case:

```text
1000 ports × 1 second = 1000 seconds
```

That's over 16 minutes.

Clearly, we need a faster approach.

#### Understanding Multithreading

Multithreading allows multiple tasks to run concurrently.

Instead of scanning ports one at a time:

```text
Port 22 → Port 80 → Port 443 → Port 8080
```

We can scan many ports simultaneously:

```text
Port 22
Port 80
Port 443
Port 8080
Port 3306
Port 8081
...
```

All running at roughly the same time. This significantly reduces the overall scan duration.

#### 1. Importing the Threading Module

The first change appears at the top of the program.

```python
from concurrent.futures import ThreadPoolExecutor
```

Python provides several ways to work with threads.

For this project, we use `ThreadPoolExecutor` because it is simple and beginner-friendly.

Its job is to:

* Create worker threads
* Assign tasks to those threads
* Manage thread execution automatically

Think of it as a manager distributing work among a group of workers.

#### 2. Updating the `scan_port()` Function

In Version 2, the function returned only a Boolean value.

```python
return result == 0
```

Version 3 changes this.

```python
return port, result == 0
```

Now the function returns:

```python
(port_number, status)
```

Example:

```python
(80, True)
```

or

```python
(23, False)
```

This makes it easier for worker threads to identify which port produced which result.

## Why Return the Port Number?

When multiple threads are running simultaneously, results may complete in a different order.

For example:

```text
Thread 1 → Port 80
Thread 2 → Port 443
Thread 3 → Port 22
```

Port 443 might finish before Port 22. Returning the port number ensures each thread knows exactly which result belongs to which port.

#### 3. Introducing the Worker Function

Version 3 adds a new function.

```python
def worker(port):
```

This function is responsible for scanning a single port.

Inside the function:

```python
port, is_open = scan_port(target_host, port)
```

The port is scanned and the result is returned.

If the port is open:

```python
if is_open:
    print(f"[+] Port {port} OPEN")
```

The result is displayed.

#### Why Use a Worker Function?

A worker function acts as the task that each thread performs. Think of a warehouse. The manager assigns boxes to workers. Each worker performs the same task:

```text
Pick up box
Move box
Report completion
```

In our scanner:

```text
Receive port
Scan port
Report result
```

Every thread executes the worker function independently.

#### 4. Creating the Thread Pool

The most important addition is:

```python
with ThreadPoolExecutor(max_workers=100) as executor:
```

This creates a thread pool containing up to 100 worker threads.

#### What Is a Thread Pool?

A thread pool is a collection of reusable worker threads. Instead of creating a new thread for every port, Python maintains a pool and distributes work among them.

Benefits include:

* Faster execution
* Reduced overhead
* Better resource management

Because we are now passing tasks to background threads instead of a standard for loop, we change how we use tqdm by wrapping it around our executor.map() generator instead

#### Understanding `max_workers`

```python
max_workers=100
```

This means up to 100 ports can be scanned concurrently.

For example:

```text
Thread 1 → Port 1
Thread 2 → Port 2
Thread 3 → Port 3
...
Thread 100 → Port 100
```

Once a thread finishes, it immediately starts scanning another port.

Choosing an appropriate value depends on:

* System resources
* Network speed
* Target responsiveness

For this project, 100 workers provides a noticeable speed improvement.

#### 5. Assigning Work to Threads

The next line distributes ports to the thread pool.

```python
executor.map(worker, ports_to_scan)
```

This tells Python:

```text
Run worker(port)
for every port
in ports_to_scan
```

Example:

```python
ports_to_scan = [22, 80, 443]
```

Becomes:

```python
worker(22)
worker(80)
worker(443)
```

The thread pool executes these tasks concurrently.

#### Why Is This Faster?

Network scanning spends most of its time waiting.

For example:

```text
Send connection request
Wait for response
```

During this waiting period, the CPU is mostly idle. Multithreading allows other scans to run while one connection is waiting for a response. This makes networking tasks an excellent use case for threading.

#### Performance Comparison

One of the biggest limitations of Version 2 was scanning speed. Because ports were scanned sequentially, large scans could take a significant amount of time.
To measure the impact of multithreading, both versions were tested against the same target while scanning ports 1–1000.

|  Version  |          Method        |     Time Taken   |
|-----------|------------------------|------------------|
| Version 2 |  Sequential Scanning   |   17m 14s 253ms  |
| Version 3 | Multithreaded Scanning |   0m 10s 875ms   |

This represents a speed improvement of approximately **95 times**, demonstrating why multithreading is commonly used in professional network scanning tools. We can see the time comparisons for both scans below:

![PowerShell benchmark results measuring the execution time of Version 2 scanning ports 1-1000 sequentially.](/assets/img/blog10/3-v2-scan-time.webp)
*Figure 3: Performance benchmark of Version 2 showing the time required to scan ports 1-1000 using a sequential scanning approach.*

![PowerShell benchmark results measuring the execution time of Version 3 scanning ports 1-1000 using multithreading.](/assets/img/blog10/4-v3-scan-time.webp)
*Figure 4: Performance benchmark of Version 3 demonstrating the significant speed improvement achieved through multithreaded scanning.*

In here we used the **measure-command**. It is a cmdlet in powershell that runs a script block or command internally and times how long the operation takes to execute, and displays a detailed report.

### **4. Building a Port Scanner in Python (Version 4) – Adding Banner Grabbing and Service Detection**

#### Introduction

In Version 3, we significantly improved the performance of our port scanner by introducing multithreading. Instead of scanning ports one at a time, the scanner could test multiple ports concurrently using Python's `ThreadPoolExecutor`.

While this made the scanner much faster, the output still had a limitation.

For example:

```text
[+] Port 22 OPEN
[+] Port 80 OPEN
[+] Port 443 OPEN
```

Although we know these ports are open, we don't know what services are actually running on them.
In Version 4, we introduce **banner grabbing**, a technique commonly used in cybersecurity and network reconnaissance to identify services running on open ports.

Instead of simply reporting that a port is open, the scanner now attempts to retrieve information from the service.

Example:

```text
[+] Port 22 OPEN | OpenSSH_9.6
[+] Port 80 OPEN | Apache/2.4.58
[+] Port 21 OPEN | FTP Server Ready
```

This provides far more useful information than simply knowing whether a port is open.

#### What's New in Version 4?

Compared to Version 3, we added:

* Banner grabbing
* Basic service identification
* A dedicated `grab_banner()` function
* Enhanced scan results
* More realistic reconnaissance capabilities

The scanner now attempts to identify services running behind open ports.

#### What Is Banner Grabbing?

Banner grabbing is a technique used to collect information from network services. Many services send an introductory message, known as a **banner**, immediately after a connection is established.

Examples include:

```text
220 FTP Server Ready
```

```text
SSH-2.0-OpenSSH_9.6
```

```text
Apache/2.4.58
```

These banners often reveal:

* Service type
* Software name
* Software version
* Operating system information

This information can be extremely valuable during network discovery and security assessments.

#### Why Is Banner Grabbing Useful?

Consider the following output from Version 3:

```text
[+] Port 22 OPEN
```

This tells us that something is listening on port 22. However, it doesn't tell us what service is running.

Version 4 provides additional context:

```text
[+] Port 22 OPEN | SSH-2.0-OpenSSH_9.6
```

Now we know:

* The service is SSH
* The server is running OpenSSH
* The version is 9.6

This information helps administrators understand their environment and helps security professionals identify potential attack surfaces.

#### 1. Introducing the `grab_banner()` Function

The biggest addition in Version 4 is:

```python
def grab_banner(host, port):
```

This function connects to an open port and attempts to retrieve any information the service provides.

If successful, the function returns the banner.

If no banner is available, it returns:

```python
"Unknown Service"
```

#### 2. Creating a New Socket

Just like our scanning function, banner grabbing requires a TCP socket.

```python
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```

This creates a new TCP connection that will be used specifically for retrieving service information.

#### 3.Setting a Timeout

```python
sock.settimeout(2)
```

The timeout is increased to two seconds.

Why?

Some services require additional time before sending a banner.
Without a timeout, the scanner could become stuck waiting indefinitely.

#### 4. Connecting to the Service

```python
sock.connect((host, port))
```

Unlike `connect_ex()`, which simply tests connectivity, `connect()` establishes a full TCP connection.
Once connected, we can interact with the service.

#### 5. Receiving Data from the Service

The most important line in the function is:

```python
banner = sock.recv(1024)
```

The `recv()` method receives data from the remote service.

The value `1024` specifies the maximum number of bytes to read.

Think of this as saying:

> "Read up to 1024 bytes of information sent by the server."

#### Understanding `recv()`

Many network services automatically send information after a client connects.

For example:

```text
220 FTP Server Ready
```

or

```text
SSH-2.0-OpenSSH_9.6
```

The `recv()` function captures this information and stores it in the `banner` variable.

#### 6. Converting Bytes to Text

Network data is transmitted as bytes.

To display it properly, we must convert it into a string.

```python
banner = sock.recv(1024).decode(errors="ignore").strip()
```

Let's break this down.

#### `decode()`

```python
.decode()
```

Converts raw bytes into readable text.

#### `errors="ignore"`

```python
.decode(errors="ignore")
```

Some services send non-text data. Ignoring decoding errors prevents the scanner from crashing.

#### `strip()`

```python
.strip()
```

Removes unnecessary whitespace and newline characters.

#### 7. Closing the Connection

After retrieving the banner, the connection is closed.

```python
sock.close()
```

Closing unused connections is important because it:

* Frees system resources
* Prevents connection leaks
* Improves overall stability

#### 8. Handling Missing Banners

Not every service provides identifying information.

Some services:

* Send nothing
* Require authentication
* Require specific commands

To handle these situations, the scanner checks:

```python
return banner if banner else "Unknown Service"
```

If data was received:

```text
SSH-2.0-OpenSSH_9.6
```

It is returned.

Otherwise:

```text
Unknown Service
```

is displayed instead.

#### 9. Error Handling

Banner grabbing can fail for many reasons.

Examples include:

* Firewall restrictions
* Connection resets
* Services that do not provide banners
* Network issues

To prevent crashes, the function uses:

```python
except Exception:
    return "Unknown Service"
```

If anything goes wrong, the scan continues normally.

#### 10. Updating the Worker Function

Version 3 only displayed open ports.

```python
if is_open:
    print(f"[+] Port {port} OPEN")
```

Version 4 adds banner retrieval.

```python
if is_open:
    banner = grab_banner(target_host, port)
    print(f"[+] Port {port} OPEN | {banner}")
```

Now the scanner:

1. Detects an open port
2. Connects to the service
3. Retrieves the banner
4. Displays the result

This makes the output significantly more informative.

#### Example Scan Results

Example output:

```text
Scanning 192.168.1.10
--------------------------------------------------
[+] Port 21 OPEN | 220 FTP Server Ready
[+] Port 22 OPEN | SSH-2.0-OpenSSH_9.6
[+] Port 80 OPEN | Apache/2.4.58
--------------------------------------------------
Scan complete
```

Compared to Version 3, we now know exactly which services are running on the target.

#### Networking Insight: Why Some Ports Show "Unknown Service"

You may notice output like:

```text
[+] Port 443 OPEN | Unknown Service
```

This does not necessarily mean the scanner failed.

Many modern services:

* Require encrypted communication
* Wait for specific requests
* Do not automatically send banners

For example:

* HTTPS servers usually expect an HTTP request before responding.
* Databases often require authentication before revealing information.
* Some administrators intentionally disable banners for security reasons.

As a result, banner grabbing works well for many services but not all of them.

## Testing Environment

To test the banner grabbing functionality, I set up a Linux machine on my local network and used a simple Python server to host several custom services with predefined banners. The server code is available in the project's GitHub repository under the testing directory.

You can view the code here: <https://github.com/Ramiyakaru/python-port-scanner/blob/main/testing/banner_server.py>

If you'd like to experiment with it yourself, simply modify the PORT value and customize the banner message before running the server. This provides an easy way to create a controlled testing environment for validating port scanning and banner grabbing functionality without relying on external services.
This allowed me to verify that the scanner could:

* Detect open ports
* Establish TCP connections
* Retrieve service banners
* Display the banner information in the scan results

You can see the scan results below.

![Terminal output from Version 4 showing open ports and retrieved service banners.](/assets/img/blog10/6-v4-output.webp)
*Figure 6: Version 4 scanner performing banner grabbing to identify services running on discovered open ports.*

Version information can help attackers identify known vulnerabilities. For this reason, many organizations disable or modify service banners. By adding service detection, our scanner now provides significantly more useful information during network reconnaissance and security assessments.

---

### **5. Building a Port Scanner in Python (Version 5) – Exporting Results to JSON**

#### Introduction

In Version 4, we enhanced our scanner by adding banner grabbing. Instead of simply identifying open ports, the scanner could also retrieve information about services running on those ports.

Example output:

```text
[+] Port 22 OPEN | SSH-2.0-OpenSSH_9.6
[+] Port 80 OPEN | Apache/2.4.58
```

While this information is useful, it disappears when the program exits.

If we want to:

* Review scan results later
* Share findings with others
* Import data into another tool
* Build reports

we need a way to save these results.

In Version 5, we introduce **JSON export functionality**, allowing scan results to be stored in a structured file for future analysis.

#### What's New in Version 5?

Compared to Version 4, we added:

* JSON support
* Structured result storage
* Automatic report generation
* Dictionaries for storing scan information
* Exporting results to a file

Example output file:

```json
[
    {
        "port": 22,
        "status": "OPEN",
        "banner": "SSH-2.0-OpenSSH_9.6"
    },
    {
        "port": 80,
        "status": "OPEN",
        "banner": "Apache/2.4.58"
    }
]
Scan complete. Results saved to scan_results.json
```

#### Why Save Scan Results?

Imagine scanning 1,000 ports:

```bash
python scanner.py 192.168.1.10 1-1000
```

The scan may discover numerous open ports and services. Once the terminal window is closed, that information is lost.

Saving scan results allows us to:

* Archive findings
* Compare scans over time
* Generate reports
* Import data into other tools
* Automate future analysis

#### 1. Importing the JSON and datetime Module

The first addition appears near the top of the script:

```python
import json
from datetime import datetime
```

Python's built-in `json` module allows us to:

* Create JSON files
* Read JSON files
* Convert Python objects into JSON format

Because JSON support is built into Python, no additional installation is required. The `datetime` module provides access to the current system date and time, which can then be formatted into a string suitable for use in a filename.

#### 2. Creating a timestamp variable

we generate a timestamp using the `strftime()` method:

```python
timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
```

#### 3. Creating a Results Container

Near the beginning of the script:

```python
scan_results = []
```

This creates an empty list. As open ports are discovered, information about those ports will be stored inside this list.

For example:

```python
[
    {
        "port": 22,
        "status": "OPEN",
        "banner": "SSH-2.0-OpenSSH_9.6"
    }
]
```

#### Understanding Dictionaries

Version 5 introduces a new Python data structure: the dictionary.
Inside the worker function:

```python
result = {
    "port": port,
    "status": "OPEN",
    "banner": banner
}
```

This creates a dictionary containing three pieces of information.
Dictionaries store data using key-value pairs:

```python
{
    "name": "Alice",
    "age": 25
}
```

In our scanner:

```python
{
    "port": 22,
    "status": "OPEN",
    "banner": "SSH-2.0-OpenSSH_9.6"
}
```

Each value is associated with a descriptive label, making the data easy to understand and process.

#### 4. Adding Results to the List

After creating the dictionary, it is added to the results list:

```python
scan_results.append(result)
```

As more open ports are discovered, additional dictionaries are added.
Example:

```python
scan_results = [
    {
        "port": 22,
        "status": "OPEN",
        "banner": "SSH-2.0-OpenSSH_9.6"
    },
    {
        "port": 80,
        "status": "OPEN",
        "banner": "Apache/2.4.58"
    }
]
```

The list grows automatically throughout the scan.

#### Why Store Data Instead of Only Printing It?

Printing information is useful for immediate feedback:

```python
print(f"[+] Port {port} OPEN")
```

However, terminal output is temporary. Structured data storage allows us to:

* Reuse results later
* Build reports
* Create dashboards
* Perform automated analysis

This concept is widely used in cybersecurity automation and software development.

#### 5. Generating Timestamped Report Files

To save scan results, we can generate a unique filename using the current date and time. We can then use the timestamp we created earlier to create a unique filename:

```python
filename = f"scan_results_{timestamp}.json"
```

Resulting in filenames such as:

```text
scan_results_20260613_115001.json
scan_results_20260613_120530.json
scan_results_20260614_091200.json
```

This approach ensures that every scan produces a new report file instead of replacing an existing one.

#### 6. Writing Scan Results to JSON

Once scanning is complete, the program writes the collected data to the newly generated file:

```python
with open(filename, "w") as f:
    json.dump(scan_results, f, indent=4)
```

The `open()` function creates the report file using the generated filename.

The `"w"` indicates that the file is being opened in **write mode**. Since every filename contains a unique timestamp, each scan generates a separate report file.

The export itself is performed using:

```python
json.dump(scan_results, f, indent=4)
```

* `scan_results` – The Python data structure containing the scan results.
* `f` – The file object where the data will be written.
* `indent=4` – Formats the JSON output using indentation to improve readability.

Without indentation:

```json
[{"port":22,"status":"OPEN"}]
```

With indentation:

```json
[
    {
        "port": 22,
        "status": "OPEN"
    }
]
```

Using timestamped filenames provides a simple way to maintain a history of scan reports, making it easier to compare results over time and archive previous scans for future reference.

#### Understanding the Output File

After a successful scan, the generated JSON file might look like this:

```json
[
    {
        "port": 21,
        "status": "OPEN",
        "banner": "220 FTP Server Ready"
    },
    {
        "port": 22,
        "status": "OPEN",
        "banner": "SSH-2.0-OpenSSH_9.6"
    },
    {
        "port": 80,
        "status": "OPEN",
        "banner": "Apache/2.4.58"
    }
]
```

You can see the scan output and the sample JSON file output below.

![Terminal output from Version 5 showing scan results and JSON export functionality.](/assets/img/blog10/7-v5-output.webp)
*Figure 7: Version 5 scanner displaying scan results before exporting the findings to a JSON report.*

![Contents of the generated JSON file containing structured scan results.](/assets/img/blog10/v5-JSON-output.webp)
*Figure 8: JSON report generated by Version 5 containing structured information about discovered open ports and service banners.*

#### Why Cybersecurity Tools Use JSON

JSON has become one of the most widely used formats in cybersecurity.

Examples include:

* Vulnerability scanners
* Threat intelligence feeds
* Cloud APIs
* SIEM platforms
* Security automation frameworks

JSON is popular because it is:

* Human-readable
* Lightweight
* Easy to parse
* Language-independent

Almost every programming language supports JSON.

By saving scan results in JSON format, the scanner becomes significantly more useful for automation, reporting, and future integrations. This final enhancement brings the project closer to the workflow used by real-world cybersecurity and network assessment tools.

### Conclusion

Across these five versions, the project evolved from a simple Python script into a fully functional network reconnaissance tool. What started as a basic port checker gradually gained real-world capabilities like range scanning, multithreading for performance, service banner grabbing, and finally structured reporting through JSON export.

This project can now serve as a foundation for further exploration into cybersecurity tooling, such as vulnerability scanning, service enumeration, or integration with threat intelligence APIs. While simple in design, it reflects the core principles behind many professional security tools used in industry today.

If you’re currently learning Python, I highly recommend building projects like this. They reinforce programming fundamentals, improve problem-solving skills, and help create a portfolio that demonstrates practical experience.

Happy coding and stay secure!
