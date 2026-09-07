# Set Up


## VS Code

## Docker

messager to Docker
```
Make a folder for this and open a terminal in it
Create a new folder anywhere, e.g. D\NYU\mit6.1810-docker. Open it in File Explorer, then either right-click empty space and choose Open in Terminal (Windows 11), or hold Shift and right-click empty space and choose Open PowerShell window here (Windows 10).

Create a file named exactly "Dockerfile"
Easiest way: open that folder in VS Code (code . in the terminal you just opened, or File → Open Folder), then File → New File, name it Dockerfile with no extension, paste this in, and save:
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y \
    openssh-server sudo git build-essential gdb-multiarch \
    qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu \
    && mkdir /var/run/sshd

RUN useradd -m -s /bin/bash mitlab && echo "mitlab:mitlab" | chpasswd && adduser mitlab sudo

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]⧉
This is the same recipe as the install step above (QEMU + RISC-V toolchain), just baked into an image instead of typed by hand, plus an SSH server so VS Code can connect the same way it connects to the CS162 Workspace.

Build the image
Back in the terminal, in that same folder:

docker build -t mit6-1810 .⧉
This downloads Ubuntu and installs everything from the Dockerfile — expect it to take a few minutes the first time.

Run it
docker run -d -p 2223:22 --name mit6-1810 mit6-1810⧉
Port 2223 is arbitrary — anything not already used by the CS162 Workspace's 16222 works. docker ps should now list mit6-1810 as running.

Add it to your SSH config
Same file you already have docker162 in — just add a second block:

Host mitvm
    HostName 127.0.0.1
    Port 2223
    User mitlab⧉
Connect from VS Code
Remote-SSH: Connect to Host → mitvm, password mitlab — identical process to connecting to docker162 earlier. Reinstall any extensions you need inside this window too; it's a separate container with its own extension host.

Clone the labs, right away — no install step needed
Everything from the install step is already baked into the image, so you can jump straight to cloning:

git clone git://g.csail.mit.edu/xv6-labs-2024
cd xv6-labs-2024
make qemu⧉
Stopping and restarting later
The container keeps running in the background even after you close VS Code. To stop it: docker stop mit6-1810. To start it again next time: docker start mit6-1810 — your files inside it persist across stop/start (only docker rm would delete them).
If you'd rather use a traditional VM instead of Docker, VirtualBox and UTM are the alternative:

VirtualBox
Windows/Intel Mac/Linux
Full isolation, its own disk image, an ISO you install yourself.

UTM
Apple Silicon Mac
Use this instead of VirtualBox specifically on M-series Macs, where VirtualBox is unreliable.

Install a hypervisor
VirtualBox or UTM.

Download Ubuntu Server 24.04 LTS and create the VM
The Server ISO, no GUI needed. ~4 GB RAM, 2 CPUs, 20 GB disk. Tick Install OpenSSH server during setup — the step people miss.

Forward a port and connect
NAT with port forwarding, host 2222 → guest 22, then the same SSH config + VS Code Remote-SSH steps as the Docker path above, followed by the manual apt-get install command from the install step since nothing is pre-baked here.

Get the lab code
Track B · MIT 6.1810
git clone git://g.csail.mit.edu/xv6-labs-2024
cd xv6-labs-2024
make qemu   # boots base xv6; Ctrl-a x to quit⧉
If the network blocks the git:// protocol
Some firewalls block port 9418. If the clone hangs, you're behind one — try it from a network that allows outbound git://, or from inside the CS162 Docker container.
Don't clone a "solutions" repo
Searching GitHub for xv6-labs-2024 turns up dozens of other students' completed forks. Cloning one of those hands you filled-in answers instead of the skeleton — it defeats the point and isn't something to lean on even for self-study. The command above pulls MIT's actual skeleton; that's the one to start from.
The per-lab workflow
Track B · MIT 6.1810
Check out the lab's branch
Each lab lives on its own branch off the skeleton (see the track below for names). Commit whatever you're working on first, then switch:

git checkout util⧉
Read the lab page, then implement
Each lab page under /labs/ on the course site spells out the exercises and hints — that's the actual assignment text, this guide is just the plumbing around it.

Grade locally, as often as you like
make grade              # everything in this lab
./grade-lab-util sleep  # one exercise at a time⧉
Skip the Gradescope step
The official flow ends with make zipball uploaded to Gradescope — that needs a class roster login you don't have. It's not a loss: make grade already runs the exact same tests and prints the same pass/fail output Gradescope would show you.

```

Stop:
```
docker stop mit6-1810



```
Restart:
```
docker start mit6-1810
```
Your files persist across stop/start cycles. 
Verify status:
```
docker ps
```

##
