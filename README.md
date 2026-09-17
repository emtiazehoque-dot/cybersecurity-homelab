# Cybersecurity Homelab

My setup for experimenting with dangerous but beautiful networks and you-know-what stuffs. I'll try to document as much as possible to track my progress.

# Setups:

- Ubuntu server using virtual box. Exact iso: `ubuntu-26.04.1-live-server-amd64.iso` [Download Link.](https://ubuntu.com/download/server)
- Virtual Box version: 7.0. Exact release: `VirtualBox Graphical User Interface Version 7.0.12_Ubuntu r159484` [Download VirtualBox.](https://www.virtualbox.org/wiki/Downloads)
- Server Resources:
  - Base memory: 2048 MB
  - Processor: 4 core
  - Video memory: 16 MB
  - Memory allocation (dynamic): 32 GB
- Network:
  - Attached to: Bridged Adaptor (helps to get independent ip for the VM)

Here's the YT video that helped me set up. [Video Link.](https://youtu.be/-mrrisBSF3A?si=Kv33VGFhxlx2-2bS)

# Day 1: Ubuntu Server Homelab Setup

**Date:** 2026-09-17

**Topics:** Ubuntu Server, SSH, HTTP Server, Avahi, SCP, Local Network Access

## 1. Initial VM Setup

I configured the Ubuntu Server virtual machine according to the previously defined setup requirements. During the installation process, I created a username and password for the server.

This VM serves as the foundation for my cybersecurity homelab experiments.

---

## 2. Checking the Server IP Address

To identify the server's IP address, I used the `ip a` command from the VM's terminal.

```bash
ip a
```

This command displays the network interfaces and their assigned IP addresses. I used the server's local IPv4 address to connect to it from my main laptop.

---

## 3. Connecting to the Server Using SSH

I connected to the Ubuntu Server VM remotely from my laptop's terminal using **SSH (Secure Shell)**.

```bash
ssh username@ip_address
```

The SSH client prompts for the user's password if password-based authentication is enabled and SSH keys have not been configured for authentication.

After successfully connecting, I could access and operate the Ubuntu server through my laptop's terminal without directly interacting with the VM screen.

![SSH Login](images/ssh%20login%20screenshot%20day%201.png)

### Key Concept

SSH provides secure remote terminal access to a server over a network. It is commonly used for Linux server administration.

---

## 4. Setting Up a Python HTTP Server

To test local network connectivity, I created an `index.html` file beforehand using the `touch` command.

```bash
touch index.html
```

I then started a basic HTTP server using Python:

```bash
sudo python3 -m http.server 80 --bind 0.0.0.0
```

![Pthon Server](images/python%20http%20server%20day%201.png)

### Explanation

- `sudo`: Runs the command with elevated privileges. Binding to port 80 generally requires elevated privileges on Linux.
- `python3 -m http.server`: Starts Python's built-in HTTP server module.
- `80`: Specifies port 80, the standard port for HTTP.
- `--bind 0.0.0.0`: Binds the server to all available IPv4 network interfaces.

The purpose of this setup was to test whether I could access a web service hosted on the Ubuntu server from another device on the local network.

> **Security note:** Python's built-in HTTP server is intended for simple file serving and testing. It is not designed to be a secure production web server. Binding to all interfaces can expose the service to other devices that can reach the server.

---

## 5. Accessing the HTTP Server Through a Browser

Initially, I accessed the HTTP server using its local IP address:

```text
http://192.168.0.102/index.html
```

This worked, but I wanted a more convenient way to access the server without manually remembering its IP address.

During this process, I discovered **Avahi**, which supports local network service discovery through mDNS (Multicast DNS).

This allowed me to access the server using a hostname ending in `.local` instead of its numerical IP address.

---

## 6. Installing and Configuring Avahi

Avahi was not preinstalled on my Ubuntu Server setup, so I installed it manually.

### Installation

```bash
sudo apt install avahi-daemon
```

### Enable and Start the Service

```bash
sudo systemctl enable --now avahi-daemon
```

### Explanation

- `avahi-daemon`: A service that enables mDNS-based local hostname resolution and service discovery.
- `systemctl enable --now`: Enables the service to start automatically at boot and starts it immediately.

Avahi allows compatible devices on the local network to resolve hostnames using the `.local` domain.

---

## 7. Accessing the Server Using a Hostname

To find the hostname of the Ubuntu Server, I used:

```bash
hostname
```

The command returned the hostname assigned to the server.

I could then access the HTTP server using the following format:

```text
http://hostname.local/index.html
```

For example, if the server's hostname is `ubuntu-test-server`, the URL would be:

```text
http://ubuntu-test-server.local/index.html
```

### Key Concept: mDNS

Multicast DNS (mDNS) allows devices on a local network to resolve hostnames without requiring a conventional DNS server.

The `.local` suffix is commonly used for mDNS hostnames.

This was a useful improvement over accessing the server through its IP address because the hostname is easier to remember and use during local network experiments.

> **Note:** `.local` resolution depends on mDNS support and network configuration on the client and server. It may not work across different network segments or on every operating system without additional support.

---

## 8. Uploading Files to the Server

I explored file transfer methods for uploading files from my laptop to the Ubuntu Server.

There are two main approaches:

1. **Command-line tools:** SCP and SFTP.
2. **Graphical tools:** FileZilla.

### 8.1. Using SCP to Upload a File

The general syntax for transferring a file using SCP is:

```bash
scp /path/to/file.txt username@hostname.local:/destination/path/
```

Example:

```bash
scp file.txt username@ubuntu-server.local:/home/username/
```

SCP transfers files over SSH. The default SSH port is **22**, unless the server is configured to use a different port.

### 8.2. Uploading a Folder Recursively

To transfer an entire folder and its contents, I used the `-r` option:

```bash
scp -r /path/to/folder username@hostname.local:/destination/path/
```

The `-r` option enables recursive copying of directories.

![Uploading Files using scp](images/scp%20working%20screenshot%20day%201.png)

### 8.3. Other File Transfer Options

- **SFTP:** An interactive file transfer protocol that operates over SSH.
- **FileZilla:** A graphical file transfer client that supports SFTP connections.

When configuring SFTP in FileZilla, I should use:

- **Protocol:** SFTP - SSH File Transfer Protocol
- **Host:** Server IP address or hostname
- **Port:** 22 (if using the default SSH port)
- **Username:** Ubuntu server username
- **Password or key:** The configured authentication method

---

## 9. Commands Used

| Command                  | Purpose                                             |
| ------------------------ | --------------------------------------------------- |
| `sudo`                   | Execute commands with elevated privileges           |
| `ls`                     | List files and directories                          |
| `cd`                     | Change the current directory                        |
| `pwd`                    | Display the current working directory               |
| `scp`                    | Transfer files over SSH                             |
| `sftp`                   | Transfer files using the SSH File Transfer Protocol |
| `ssh`                    | Establish a remote SSH connection                   |
| `ip a`                   | Display network interfaces and IP addresses         |
| `hostname`               | Display the server's hostname                       |
| `systemctl`              | Manage and inspect system services                  |
| `touch`                  | Create a file or update its timestamps              |
| `python3 -m http.server` | Start a basic Python HTTP server                    |
| `apt install`            | Install software packages on Ubuntu                 |

---

## 10. Key Takeaways

Through this lab, I practiced:

- Setting up and configuring an Ubuntu Server virtual machine.
- Identifying the server's local IP address.
- Connecting remotely using SSH.
- Hosting a basic HTTP service using Python.
- Accessing the server through a web browser on the local network.
- Installing and configuring Avahi for local hostname resolution.
- Transferring files between my laptop and server using SCP.
- Understanding the role of ports in SSH, HTTP, and SFTP.

This setup provides a foundation for future cybersecurity experiments involving Linux administration, networking, service discovery, and security testing.
I'm planning to work on this server for my _tests_.
