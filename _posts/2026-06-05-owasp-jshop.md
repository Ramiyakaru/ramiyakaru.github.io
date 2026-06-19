---
title: "Installing OWASP Juice Shop on Proxmox"
date: 26-06-05 +1030
categories: [cybersecurity, Networking, hacking]
tags: [cybersecurity, owasp, github, proxmox]
---

## Installing OWASP Juice Shop on Proxmox Using LXC Containers

### Introduction

One of the best ways to learn cybersecurity is through hands-on practice in a safe and controlled environment. Rather than experimenting on production systems, cybersecurity professionals use intentionally vulnerable applications that are designed for learning, testing, and skill development.

One of the most popular platforms for this purpose is **OWASP Juice Shop**.

### What is OWASP Juice Shop?

OWASP Juice Shop is an intentionally vulnerable web application developed by the **OWASP (Open Worldwide Application Security Project)** community. It simulates a realistic online shopping website containing numerous security vulnerabilities that range from beginner-level flaws to advanced attack scenarios.

The application is designed to help students, cybersecurity enthusiasts, penetration testers, and security professionals learn:

* Web application security testing
* Vulnerability assessment techniques
* Penetration testing methodologies
* Secure coding practices
* Bug bounty hunting skills

The platform includes vulnerabilities from the:

* OWASP Top 10
* Broken authentication
* SQL Injection
* Cross-Site Scripting (XSS)
* Cross-Site Request Forgery (CSRF)
* Insecure Direct Object References (IDOR)
* Sensitive data exposure
* Authentication bypass
* Security misconfigurations

Unlike traditional training labs, Juice Shop provides realistic attack scenarios that closely resemble vulnerabilities found in real-world applications.

### Why Use OWASP Juice Shop?

OWASP Juice Shop is an excellent learning platform because:

* Completely free and open source
* Safe environment for testing attacks
* Contains vulnerabilities of varying difficulty levels
* Frequently updated by the OWASP community
* Supports self-paced learning
* Can be integrated into home labs and cyber ranges
* Ideal for learning tools such as Burp Suite, OWASP ZAP, Nmap, SQLMap, Nikto, and Metasploit

For anyone building a cybersecurity home lab like ourselves, OWASP Juice Shop is one of the first vulnerable applications worth deploying.

### Understanding Containers Before Installation

In this tutorial, OWASP Juice Shop will be deployed inside a **Proxmox LXC Container** rather than a traditional Virtual Machine.
Before starting, it is important to understand what containers are and why they are useful.

### What is a Container?

A container is a lightweight virtualization technology that allows applications to run in isolated environments while sharing the host operating system's kernel.
Think of a container as:

> A small self-contained package that includes everything an application needs to run except the operating system kernel.

Each container contains:

* Its own filesystem
* Applications
* Libraries
* Dependencies
* Configuration files

However, unlike a Virtual Machine, a container does not require a complete guest operating system. This significantly reduces resource consumption while maintaining isolation between applications.

### How is a Container Different from a Virtual Machine?

### Virtual Machine

A Virtual Machine emulates an entire computer system.

Each VM includes:

* Virtual hardware
* Full operating system
* Applications
* Libraries

Example:

```text
Physical Server
│
├── Hypervisor (Proxmox)
│
├── Windows VM
│   └── Full Windows OS
│
├── Kali Linux VM
│   └── Full Linux OS
│
└── Ubuntu VM
    └── Full Linux OS
```

Each VM consumes:

* More RAM
* More CPU resources
* More storage space
* Longer boot times

### Container

Containers share the host operating system kernel.

Example:

```text
Physical Server
│
├── Proxmox Host
│
├── LXC Container 1
│   └── OWASP Juice Shop
│
├── LXC Container 2
│   └── Pi-hole
│
└── LXC Container 3
    └── Docker Services
```

Containers provide:

* Fast startup times
* Lower RAM usage
* Smaller disk footprint
* Efficient resource utilization

### When Should Containers Be Used?

Containers are ideal when:

* Hosting web applications
* Running Docker services
* Deploying development environments
* Creating lightweight lab systems
* Running network services
* Testing applications

Common examples include:

* OWASP Juice Shop
* Pi-hole
* Nginx
* Apache
* Grafana
* Prometheus
* Home Assistant

### Why Use a Container for OWASP Juice Shop Instead of a Virtual Machine?

Juice Shop itself is simply a web application running inside Docker. Using a full Virtual Machine would introduce unnecessary overhead because:

* A complete operating system is not required
* Juice Shop consumes minimal resources
* Containers start much faster
* Containers require less storage
* Easier to manage within Proxmox

For home lab environments where resources are limited, running Juice Shop inside an LXC container is usually the most efficient approach.

### Downloading an Ubuntu Container Template

Log in to your Proxmox VE web interface. Select your local storage from the left navigation pane. This is usually named **local**.

![Accessing Proxmox Local Storage](/assets/img/blog11/1-accessing-local-storage.webp)
*Figure 1: Selecting the local storage location in Proxmox.*

Next, select **CT Templates** and click **Templates**.

![Accessing Container Templates](/assets/img/blog11/2-accessing-templates.webp)
*Figure 2: Opening the container template management interface.*

Search for Ubuntu and download either:

* Ubuntu 22.04 LTS
* Ubuntu 24.04 LTS

Both versions work well for this project.

![Downloading Ubuntu Template](/assets/img/blog11/3-downloading-ubuntu-template.webp)
*Figure 3: Downloading an Ubuntu LXC template.*

### Creating the Ubuntu Container

After the template download completes, click **Create CT**.

![Creating Container](/assets/img/blog11/4-creating-container.webp)
*Figure 4: Starting the LXC container creation wizard.*

## General Configuration

Configure the following:

* Container ID (CT ID)
* Hostname
* Root Password

For this deployment, uncheck:

```text
Unprivileged Container
```

This simplifies Docker deployment inside the container.

## Template Selection

Choose the Ubuntu template downloaded earlier.

## Disk Configuration

Allocate:

```text
10–15 GB
```

This provides sufficient storage for Ubuntu, Docker, and Juice Shop.

## CPU Configuration

Allocate:

```text
1–2 CPU Cores
```

Juice Shop is lightweight and does not require significant processing power.

## Memory Configuration

Allocate:

```text
1024 MB (1 GB RAM)
```

This is adequate for a small lab deployment.

## Network Configuration

Select:

```text
Bridge: vmbr0
```

For IP configuration choose either:

* DHCP
* Static IP Address

A static IP is generally recommended for lab services.

## Understanding Nesting

Navigate to:

```text
Options → Features → Edit
```

Enable:

```text
Nesting
```

### What is Nesting?

Nesting allows a container to run containerization technologies such as Docker inside itself. Normally, LXC containers are restricted from creating additional namespaces and container environments. Since OWASP Juice Shop will run inside Docker, Docker needs the ability to create its own isolated container environment.

Without enabling Nesting:

* Docker may fail to start
* Containers may not launch correctly
* Permission errors can occur

Enabling Nesting allows Docker to function properly inside the LXC container.

![Container Setup Summary](/assets/img/blog11/5-container-setup-summary.webp)
*Figure 5: Reviewing the container configuration before creation.*

### Starting the Container

Once the container has been created:

1. Start the container.
2. Open the Console tab.
3. Log in using:

```text
Username: root
Password: <password configured during setup>
```

### Updating Ubuntu

Update package repositories and upgrade installed packages.

```bash
# Update the package list and upgrade system packages
apt update && apt upgrade -y
```

Keeping the operating system updated ensures the latest security patches and package versions are installed.

### Installing Docker

Install prerequisite packages.

```bash
# Install prerequisite packages
apt install -y curl apt-transport-https ca-certificates software-properties-common
```

Download and install Docker using the official convenience script.

```bash
# Download and install Docker using the official convenience script
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

After installation, Docker should be running automatically.

### Deploying OWASP Juice Shop

Run the following command:

```bash
docker run -d \
--name juiceshop \
--restart always \
--security-opt apparmor=unconfined \
-p 80:3000 \
bkimminich/juice-shop
```

### Understanding the Docker Command

#### `-d`

Runs the container in detached mode. This means the application runs in the background while returning control to the terminal.

#### `--name juiceshop`

Assigns a friendly name to the container.

This makes container management easier.

#### `--restart always`

Ensures the container automatically starts after:

* Reboots
* Docker service restarts
* Unexpected crashes

#### `-p 80:3000`

Maps network ports.

```text
Host Port      Container Port
80      --->   3000
```

Users access:

```text
http://container-ip
```

while Juice Shop internally listens on port 3000.

#### `--security-opt apparmor=unconfined`

This is particularly important when running Docker inside an LXC container.

AppArmor is a Linux security module that restricts application capabilities.

When Docker runs inside certain Proxmox containers, AppArmor restrictions may prevent Docker containers from starting correctly.

The option:

```text
apparmor=unconfined
```

disables AppArmor confinement for this specific Docker container, allowing Juice Shop to run without encountering security profile restrictions.

### Manually Starting Juice Shop

If required, you can manually start the container using:

```bash
docker start juiceshop
```

### Verifying Docker is Running

Check the Docker service status.

```bash
systemctl status docker
```

A healthy installation should show Docker as active and running.

![Checking Docker Status](/assets/img/blog11/6-check-docker-status.webp)
*Figure 6: Verifying that the Docker service is running.*

### Verifying Juice Shop is Running

Check active Docker containers.

```bash
docker ps
```

You should see:

* Container Name
* Container ID
* Status
* Port Mapping

The output should indicate that port 80 is mapped to port 3000 inside the Juice Shop container.

![Checking Docker Ports](/assets/img/blog11/7-check-docker-port.webp)
*Figure 7: Confirming the Juice Shop container is running and listening on the expected port.*

### Finding the Container IP Address

Display network interface information.

```bash
ip a
```

Locate the IP address assigned to the container.

![Checking Container IP Address](/assets/img/blog11/8-check-docker-ip-config.webp)
*Figure 8: Identifying the container IP address.*

### Accessing OWASP Juice Shop

From another device on the same network, open a web browser and navigate to:

```text
http://<container-ip>
```

Example:

```text
http://10.10.20.61
```

If everything has been configured correctly, the OWASP Juice Shop homepage will load.

![OWASP Juice Shop Homepage](/assets/img/blog11/9-install-complete.webp)
*Figure 9: Successfully accessing OWASP Juice Shop from a web browser.*

### Conclusion

In this tutorial, we deployed OWASP Juice Shop inside a Proxmox LXC container using Docker. By leveraging containers instead of full Virtual Machines, we significantly reduced resource usage while maintaining a realistic environment for web application security testing.
With Juice Shop successfully installed, you now have a vulnerable web application that can be used to practice:

* Burp Suite testing
* OWASP ZAP assessments
* SQL Injection attacks
* Cross-Site Scripting (XSS)
* Authentication bypass techniques
* Web application enumeration
* Secure coding analysis

As your cybersecurity lab grows, OWASP Juice Shop can be integrated alongside platforms such as Metasploitable, DVWA, Kali Linux, and pfSense to create a comprehensive penetration testing environment for learning and experimentation.

Hapy Hacking and Stay Secure!!
