# RPI4B k8s Cluster deploy step-by-step software tutorial

## Quick Data Reference Guide

This is the planned network component for this sample RPi4B k8s cluster.
RPi4B MAC|Static IP|Hostname|SSH Command
-|-|:-:|-
dc:a6:32:00:00:01|192.168.101.101|rpi01|`$ ssh ubuntu@192.168.101.101`
dc:a6:32:00:00:02|192.168.101.102|rpi02|`$ ssh ubuntu@192.168.101.102`
dc:a6:32:00:00:03|192.168.101.103|rpi03|`$ ssh ubuntu@192.168.101.103`
dc:a6:32:00:00:04|192.168.101.104|rpi04|`$ ssh ubuntu@192.168.101.104`
...|...|...|...

---

## Card preparation per RPi4B Node

#### This section contains the process that must be done per card\node using linux in your preparation machine

1. Install **Raspberry Pi Imager**

> https://www.raspberrypi.org/downloads/

2. Run **Raspberry Pi Imager** and burn the latest **Ubuntu LTS 64bit** version into the card:

> ie.: `Ubuntu 20.04 LTS (Pi3/Pi4) 64bit`

3. Safelly, unmount the card and place it in the RPi4B

---

## Setting up the basics

1. Have a router with internet connection
2. Connect your client machine to the same network provided by that same router (via eth or WiFi)
3. Connect the RPi4B to a router via RJ-45 (Cat-5e or later)
4. With the client machine, access the router interface and set a manual override of the IP based on the MacAddress of the RPi4B (AKA Static IP over Mac Address). *Look for help in your router's manual on how to do this.*

```shell
ie.
    Mac : dc:a6:32:00:00:01
    Ip  : 196.168.101.101
```

5. Reboot the router and make sure the setting stayed as you configured them.

---

## RPi4B Node First-boot

1. Power-ON the RPi4B and wait around 80secs (it can take more or less depending on the quality of your card)
2. Once the RPi4B is fully booted, open a terminal in your client machine
3. Type:

```shell
ssh ubuntu@192.168.101.101
```

4. A message will return:

```
The authenticity of host '192.168.101.101 (192.168.101.101)' can't be established.
ECDSA key fingerprint is SHA256:l5FbmYgBKyAem+K+aQihph/vDlDFNoTSoReA1HI2MHFc.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

5. Type `yes` and press ENTER key
6. Ubuntu's official image, default credentials are as follow:

```
Username: ubuntu
Password: ubuntu
```

7. After login, it will force you to change password for increased security:

```
WARNING: Your password has expired.
You must change your password now and login again!
Changing password for ubuntu.
```

8. Follow the indicated steps:

```
Current password: ubuntu
New password: it-can-be-whatever-you-want
Retype new password: >>reenter-the-new-password<<
```

9. The change should be successfull but will force a close of the ssh session.

---

## Preparing and configuring the RPi4B Node

1. Connect to the RPi4B via ssh with:

```shell
ssh ubuntu@192.168.101.101
```

2. Login with your new password

3. Now you are in the RPi4B Ubuntu's terminal and should see a prompt like this:

```
ubuntu@ubuntu:~$
```

4) Type the following commands:

```shell
echo "clear && sudo apt update && sudo apt-get upgrade -y && sudo apt autoremove && sudo apt-get clean && sudo apt-get --purge autoremove -y && snap refresh --list && sudo snap refresh" > update.sh
chmod +x update.sh
echo "alias update='~/update.sh'" >> ~/.bash_aliases
source ~/.bashrc
update
```

4. Navigate to the following directory:

```shell
cd /boot/firmware
```

5. Edit the file `nobtcmd.txt` or `cmdline.txt` depending on your Ubuntu image version:

```shell
sudo vim cmdline.txt
```

**You should see something like this:**
```ini
net.ifnames=0 dwc_otg.lpm_enable=0 console=ttyAMA0,115200 console=tty1 root=LABEL=writable rootfstype=ext4 elevator=deadline rootwait fixrtc
```

6. There should be a single line of commands so at the ***very begining*** of that line add:

```ini
cgroup_enable=memory cgroup_memory=1 
```

7. Save the file and exit the editor

8. **Optional** Add all Rpi Nodes to the 'hosts'

```shell
sudo vim /etc/hosts
```

Then add the following and save:

```shell
192.168.101.101 rpi01
192.168.101.102 rpi02
192.168.101.103 rpi03
192.168.101.104 rpi04
```

9. Reboot node

```shell
sudo reboot
```

10. **Optional** You can check (after reboot) with the command on the Node:

```shell
cat /proc/cgroups
```

---

## Passwordless RPi4B Login

#### On your client machine ( *NOT a RPi4B* )

1. Generate a new RSA key especific for your RPi4B's

```shell
mkdir ~/.ssh/rpi
ssh-keygen -f ~/.ssh/rpi/rpi4-key -t rsa -b 4096
```

2. On:

```
Enter passphrase (empty for no passphrase):
```
...just press ENTER 2x.

4. Now make it available on your client machine with:

```shell
echo "IdentityFile ~/.ssh/rpi/rpi4-key" >> ~/.ssh/config
```

5. It's time to copy the public key to the RPi4B Node with:

```shell
ssh-copy-id -i ~/.ssh/rpi/rpi4-key.pub ubuntu@192.168.101.101
```

6. From now on, connecting via SSH shouldn't require password if connected through your client machine.

---

## Static IP, Hostname and other RPi4B stuff

#### On your client machine connected to the RPi4B via SSH

1. **[Optional]** You can locally configure a static IP for the RPi4B Node you're working with using the command:

```shell
sudo ifconfig eth0 192.168.101.101 netmask 255.255.255.0
```

2. Giving a good hostname makes things easier, so to set the hostname use the command:

```shell
sudo hostnamectl set-hostname rpi01
```

3. Run an update command and reboot the RPi4B to make sure everything is set and ready:

```shell
update
sudo reboot
```

---

> From this point on you either do all the previous steps for each cluster node you want to add and then install the k8s in each, or you go creating it all from top to bottom one by one. Your choice.

---

## Finally, MicroK8s

1. You should be able to connect to the pre-configured cluster node by it's hostname instead of the IP address:

```shell
ssh ubuntu@rpi01
```

2. Make sure everything it up-to-date:

```shell
update
```

3. Install the Microk8s the easy way via Snap:

```shell
sudo snap install microk8s --classic
```

4. Let's make the user run Microk8s without `sudo` command:

```shell
sudo usermod -a -G microk8s ubuntu
sudo chown -f -R ubuntu ~/.kube
```

5. Add some usefull alias:

```shell
ie.:
echo "alias kubectl='microk8s.kubectl'" >> ~/.bash_aliases
```

6. Reboot before using...

```shell
sudo reboot
```

7. Now let's check if everything is ok...

```shell
microk8s.status
kubectl cluster-info
kubectl get no
kubectl describe node rpi01
```
8. Add slave nodes to master node

* For each slave node:
```shell
master@rpi01$ microk8s add-node
Join node with: microk8s join 192.168.101.101:25000/< redacted >

If the node you are adding is not reachable through the default interface you can use one of the following:
 microk8s join 192.168.101.101:25000/< redacted >
 microk8s.join 10.1.23.0:25000/< redacted >
```
```shell
slave@rpi02$ microk8s join 192.168.101.101:25000/< redacted >
```
This could take some time.

9. Check if node was added

```shell
master@rpi01$ microk8s kubectl get node
NAME    STATUS     ROLES    AGE   VERSION
rpi01   Ready      <none>   18d   v1.18.2
rpi02   Ready      <none>   18d   v1.18.2
```

10. And after all Nodes were added

```shell
master@rpi01$ microk8s kubectl get node
NAME    STATUS     ROLES    AGE   VERSION
rpi01   Ready      <none>   18d   v1.18.2
rpi02   Ready      <none>   18d   v1.18.2
rpi03   Ready      <none>   18d   v1.18.2
rpi04   Ready      <none>   18d   v1.18.2
...
```
