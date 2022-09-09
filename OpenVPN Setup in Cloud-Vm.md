# OpenVPN Setup in Cloud-Vm

First we need cloud instance. We have many options like AWS, Azure, Google cloud, Digital ocean, Linode, etc. Just create Ubuntu VM with a free tier, and allow SSH (22), http (80), https (443) ports. 



![](https://i.imgur.com/XB68rIP.png)

Next, In Networking tab under public ip box click on "create new" then a dialog box will appear on the right side :



![](https://i.imgur.com/zQYVarq.png)



In Assignment, change it from dynamic to Static and hit ok :



![](https://i.imgur.com/6cTLVMr.png)



Continue next till review & create, review your VM and create it. After deploying VM, go to networking tab :



![](https://i.imgur.com/UHNnC4c.png)



Click on Add Inbound Port Rules :



![](https://i.imgur.com/2qPR2WV.png?1)



Add Inbound port rules :

| Source | Destination | Permission |
| ------ | ----------- | ---------- |
| Any    | 943/tcp     | Allow      |
| Any    | 1194/udp    | **Allow**  |

> **It is very important to allow these ports as they are default ports used by OpenVPN access server. If you will not allow them then your client will not be able to connect to the VPN server.**



![](https://i.imgur.com/WBOR8Ir.png)



First Port you have to add is 943, type it in destination port ranges and choose protocol as TCP. Then Scroll down and write name of the port rule and priority value. Then hit on Add button.



![](https://i.imgur.com/NFtNMfj.png)

![](https://i.imgur.com/JQD1Jtf.png)



After that you have to add second port which is UDP port 1194, type 1194 in destination port ranges and select protocol UDP. Scroll down and give priority value and name for the port rule and hit Add button.



![](https://i.imgur.com/cIfSIOV.png)



Now you can see the inbound port rules added in the list along with other ports.



![](https://i.imgur.com/mWSrchK.png)



Get back to Overview tab and there you can that you have public IP address of the machine, copy it.

Open your terminal/cmd/PowerShell.
Log into your machine via SSH :

```
$ ssh user@public_ip_address
```

In place of, 
user = your username which you have set while creating the VM.
public_ip_address = public ip address of your VM.

If port of ssh is defined is different then :

```
# ssh -P port_number user@public_ip_address
```

First, update and upgrade your repo :

```
$ sudo apt update
$ sudo apt upgrade
```

Setup the password for the root :

```
$ sudo passwd root
```

You need to download the GitHub script of OpenVPN access server, for more guide you can visit https://github.com/Nyr/openvpn-install 

```
$ wget https://git.io/vpn -O openvpn-install.sh
```

![](https://i.imgur.com/keqcm8I.png)

Then you have to run the script, If you are using CMD/PowerShell then :

```
$ sudo bash openvpn-install.sh
```

![](https://i.imgur.com/OuSSD7q.png)

But if your using Linux Terminal then, first you may have to make it executable first and then run it :

```
$ sudo chmod +x openvpn-install.sh
$ sudo ./openvpn-install.sh
```

![](https://i.imgur.com/eKOfKF8.png)



Then you will be prompted with the installer welcome!
Here you will find the first option about your ip address, put in your VM's public ip address and hit enter. Then you will be asked about the port number where you want your server to listen on, you can keep it default 1194 as we have allowed that port earlier so no need to change it and hit enter. Then, choose the DNS of your choice, Best DNS from the security perspective is of Cloudflare (1.1.1.1), so will recommend in using that otherwise, there are tons of options you can choose whichever you are comfortable with. Then you will be asked the first client name. Give it a name and your client configuration file will be created.

![](https://i.imgur.com/IzNxoD3.png)

![](https://i.imgur.com/ijpxPUf.png)



In the picture above, you can see that configuration file of the first client is saved under /root/axer.ovpn
So we can copy it to the home folder with :

```
# sudo cp /root/axer/axer.ovpn /home/axer
```

After doing this, you need to check the status of your OpenVPN server, if its running or not, to do so :

```
$ systemctl status openvpn
```

If its not running then :

```
$ systemctl restart openvpn
```

![](https://i.imgur.com/sDsucj0.png)

Then you will see that OpenVPN service will start running. 

Next check on the Linux Firewall status :

```
$ ufw status
```

If its inactive, then enable it :

```
$ sudo ufw enable
```

then you'll see that its status is active now.



![](https://i.imgur.com/F9fPp06.png)



After that allow some ports :

```
$ sudo ufw allow 22/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 443/tcp
$ sudo ufw allow 943/tcp
$ sudo ufw allow 1194/udp
```

After doing this, if you want to add more user's/clients to the VPN or u want to revoke any client then :

```
$ sudo ./openvpn-install.sh
```

run this script again, you'll get options to do and even to delete the OpenVPN server as well.



![](https://i.imgur.com/NeSSglE.png)

Type :
1 to add new client,
2 to revoke an existing client,
3 to remove OpenVPN from the server,
4 to exit from the script.

After this if you want no logs on the server then you can checkout these two files for logs :

```
/usr/local/openvpn_as/etc/db/log.db
/var/log/openvpnas.log
```

You can edit these log files in order to save logs or not.

Now, in the end you have to transfer that .ovpn config files to your local machine, for that we can use **`scp`** , open command prompt if your on windows machine and type in :

```
# scp -P 22 user@public_ip_address:/home/user/client1.ovpn .
```

here, P = port number, dot (.) in the end is important in order to tell the machine to download the file in that respective folder where we are, so you should navigate yourself to the specific folder before executing that command.

![](https://i.imgur.com/aF3gVoz.png)

