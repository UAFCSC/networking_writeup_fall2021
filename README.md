# UAF CSC Networking Lab Writeup

Dylan Palmieri

## First Steps

The first step in a new network that you don't know anything about
should always be to figure out what you *do* know. In this case,
I'd made a doc with a table for everyone to put their name, ip address (`ip addr`),
operating system, open ports (`ss -tulpan`), and comments. From there, using the
`nmap -PR 10.250.101.*` command, we were able to run an ARP scan over the
entire subnet to determine what other machines were open and what they were
running. We were able to identify three boxes: 2 Linux and 1 Windows.

## Box 1: Linux (10.250.101.215)

### Open ports: 21 (FTP), 22 (SSH)

One good note about this box is that when you try to SSH to it, it gives you
a message like "Public Key Denied" without ever prompting for a password. This
means that the person that configured the SSH service on this machine set it up
so that it only accepts
[SSH keys](https://www.geeksforgeeks.org/introduction-to-sshsecure-shell-keys/).
Without a key it is computationally infeasible to brute-force entry into this box,
so the next step would be to check out the FTP server that is running.

In the `nmap` output, it told us specifically that anonymous FTP was enabled for
this server, which you can use by entering "anonymous" as your username when you
connect to the FTP server (`ftp 10.250.101.215`).
