# UAF CSC Networking Lab Writeup

Dylan Palmieri

## First Steps

The first step in a new network that you don't know anything about
should always be to figure out what you *do* know. In this case, I'd made
[a doc](https://docs.google.com/document/d/1eKIDQgEndI37_XJpw4qg7oQKK7fPAHw1gozDS3LOTqo/edit?usp=sharing)
with a table for everyone to put their name, ip address (`ip addr`),
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
connect to the FTP server (`ftp 10.250.101.215`). When connecting to the FTP server
in "anonymous" mode, a message pops up at the beginning of the FTP screen, "Welcome
to jdoe's Dotfiles server!". It appears that our friend jdoe is sharing their
user configuration files, and if you look at them with `ls -alh`, you'll see that
there's a ".ssh" directory. This is typically not a part of dotfile repos, as it
usually contains __SSH keys__, which is the only way of getting SSH on this box!

If you download the SSH keys and get to the box, there's a script in the user's
home directory that has one command: `apt-get update && apt-get upgrade -y`. The
interesting thing about this command is that usually apt requires `sudo` privileges,
and in this case there's no `sudo` present, which means that this command will
only work if the root user runs it. If you change the command and
`cat /etc/shadow > /home/jdoe/shadow.txt`, in about one minute you should see
a text file appear on the screen with a flag at the bottom!

## Box 2: Linux (10.250.101.223)

### Open Ports: 22 (SSH)

## Box 3: Windows Server 2019

## Open Ports: 135, 139, 445, 5357

I'm not a Windows guy (and I'll admit to being a bit short on time when I was
putting this lab together), so I didn't actually add any path to get root (or
Admin, whatever) on this box. In hindsight, I should have enabled SMB 1.0 and
you could have used Metasploint + Eternal Blue to get in.
