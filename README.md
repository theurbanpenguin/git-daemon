# Configuring a Git Server Using Git Daemon

---

> In this module you will learn to use the git server using the git protocol. 
> This will set up the git server in an **unauthenticated mode**. This is useful
> perhaps for a public read-only git server or a private departmental git server
> without access to a public network. You will also learn how to configure 
> both read-write access and how to export repositories globally ot selectively.
> 
> The demo uses to Rocky Linux 8.10 systems but the basis of the operation is
>  transferable to other Linux systems.
> 
> The video is on [YouTube](https://youtu.be/f9HlmOJo98I)
> 
> The git repo is [here](https://github.com/theurbanpenguin/git-daemon)

## Working on the Git Server 
```bash
# On Rocky 1: The Git Server
sudo yum install -y git git-daemon
sudo useradd -m -d /srv/git
# Open interactive shell as the git user
sudo -u git -i
git init --bare example.git
exit
# Create the service unit
sudo systemctl edit --full --force git-server
```
### Define the Service Unit
```text
# In the editor
[Unit]
Description=Git Daemon Service
After=network.target

[Service]
ExecStart=/usr/bin/git daemon --export-all --enable=receive-pack --reuseaddr --base-path=/srv/git --verbose
Restart=always
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=git-daemon

[Install]
WantedBy=multi-user.target

#Save the file
```
### Start the service and add firewall rule
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now git-server

# Check for port 9418
ss -ntl

sudo firewall-cmd --add-service=git
sudo firewall-cmd --runtime-to-permanent
```
## Working on Git Client
Now move to the Rocky 2 system, you git client. Assuming the git client
is installed and the git user.name and email is set
```bash
git clone git://172.16.247.160/example
cd example
touch README.md
git add .
git commit -m "First commit"
git push
```
### Selectively Export Repo

Rather than exporting all repos with the option **--export-all**, you can add
a file **git-daemon-export-ok** to the root level of the repo:

```bash
sudo -u git touch /srv/git/example.git/git-daemon-export-ok
```