Table of Content 
[](https://cs162.org/static/hw/hw-intro/docs/executable/)

# Set up


## VS Code

## Docker
Write in Docker
``` 
Help you set up Docker to work with this repository instead AND Clone the workspace repo
git clone https://github.com/Berkeley-CS162/cs162-workspace.git cd cs162-workspace⧉
Build and start it once, in the foreground
Wait for the line Docker workspace is ready!, then stop it with Ctrl+C.

docker-compose up⧉
Start it again, this time in the background
docker-compose up -d⧉
Sanity-check it over SSH
Password is workspace.

ssh workspace@127.0.0.1 -p 16222⧉
Optional: name it so you don't retype the port
Add to ~/.ssh/config:

Host docker162 HostName 127.0.0.1 Port 16222 User workspace⧉
Then it's just ssh docker162 from anywhere, including VS Code.
```
To close Docker
```
docker-compose down
```
To restart it later, just run

```
docker-compose up -d
```
### use VS Code Remote SSH:

Open VS Code
Press Ctrl+Shift+P
Search "Remote-SSH: Connect to Host"
Select docker162
Enter password: workspace

# HW 0

#
