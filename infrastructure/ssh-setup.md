# SSH setup

1. Change port for openssh-server service activated by socket

For newest openssh-server, Port and ListenAddress options are not used when sshd is socket-activated. Changing these two fields in /etc/ssh/sshd_config won't affect anything.

Change this file instead, it will config ssh.socket ports `sudo nano /lib/systemd/system/ssh.socket`

```
[Socket]
ListenStream=22
ListenStream=2222
Accept=no
```

### What does it mean by service activated by socket?
In the context of systemd, "activated by socket" refers to a unit that is started automatically when a specific socket is opened. A socket is a communication endpoint that allows processes to communicate with each other, either on the same machine or over a network.

When a socket is opened, systemd can automatically start a corresponding service or other unit that is configured to be activated by that socket. This can be useful for starting a service only when it is actually needed, rather than having it run constantly in the background.

```
sudo systemctl list-sockets
```

2. [Allow port 22 on LAN only. Allow port 2222 on WAN only](https://www.techrepublic.com/article/how-to-allow-ssh-connections-from-lan-and-wan-on-different-ports/)
   
3. [Setup no password for ssh](https://www.cyberciti.biz/faq/how-to-disable-ssh-password-login-on-linux/)

4. restart for new changes

```
sudo systemctl daemon-reload
sudo systemctl restart ssh
```
