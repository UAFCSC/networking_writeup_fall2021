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

I'm going to apologize in advance for this box: I had a plan which was conceptually
good, but a lack of time and proper preparation on my part meant that the plan and
reality didn't quite line up.

The first thing to do is check out the ports: we can see that this box
is running SSH, and when we try to connect to it we get prompted for a password!
The next thing to do is try to find out how we can connect. One approach is to
use Hydra, a brute-forcing tool included with Kali Linux. Another great resource
included with Hydra are password lists - common passwords that are compiled from
real-world incidents. Hydra also can brute force usernames, which is pretty cool.

This is one of the scuffed parts: I assumed that bruteforcing a short, low complexity
username (jdoe) and password (passport) would take ~5 minutes. I'm going to blame
this on tired-brain me, because that's definitely not the case. I should have
added some other kind of challenge (OSINT would be fun) to help guess with this
instead of relying on bruteforcing.

Once we've made it onto the machine, jdoe has nothing in his home directory. But
gosh darn it, our goal is to get root, so that's what we're going to do!

After looking around for a bit, you may notice that `nmap` on this box has the
SUID bit set.

[See more here about exploiting SUID](https://attackdefense.com/challengedetails?cid=76).

Whenever you see a binary is potentially vulnerable, the two most obvious ways
being the SUID and if the binary can be run as sudo on your account (always check
`sudo -l`!), your first thought should always be [GTFOBins](https://gtfobins.github.io/).
At the GTFOBins entry for `nmap`, there's a nice section on SUID and sudo, which
tells you that if you run the command `nmap --interactive`, then run `!sh` in the
interactive console, you can spawn a root shell!

This is the second scuffed part: the `--interactive` flag for `nmap` is only available
for versions between 2.02, to 5.21, and the version that we were running was 7.91.
I think that when I was looking for ways to exploit `nmap` with SUID, I looked at
a resource that was out of date without realizing it. The lesson that I'm taking
away from this is: if you're setting up a puzzle or challenge that requires research,
make sure that you follow the same path that you would if you were solving the
puzzle. In this case, that means that after I decided to set the SUID flag on nmap,
I should have checked GTFOBins for the exploit, since that's what I reccommend
people use in scenarios like this.

## Box 3: Windows Server 2019

### Open Ports: 135, 139, 445, 5357

I'm not a Windows guy (and I'll admit to being a bit short on time when I was
putting this lab together), so I didn't actually add any path to get root (or
Admin, whatever) on this box. In hindsight, I should have enabled SMB 1.0 and
you could have used Metasploint + Eternal Blue to get in.

## Closing Thoughts

Thanks for reading! I definitely learned a lot and had fun setting up this lab,
and if you have any ideas for labs in the future, are interested in giving a
lecture yourself, or just want to chat about neat security stuff, feel free
to send me a message on Discord or shoot me an email!
