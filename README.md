# Host a BitcoinXT node on Ubuntu Server 14.04 LTS / Microsoft Azure #

Motivation: To create comprehensive instructions for anyone to follow to allow users to deploy a full BitcoinXT node permanently on using Microsoft's Azure.

### Requirements ###
1. Secondary drive large enough for the block chain growth
2. No wallet
3. No UI
4. Allow to be discovered by opening firewall
5. Good uptime

Start from the Microsoft Azure portal https://portal.azure.com

### Create an Ubuntu Server VM on Azure ###

1. Select New -> Compute -> Ubuntu Server 14.04 LTS -> Create
2. Enter Host Name, e.g. BitcoinXT-2
3. Enter User name, e.g. bitcoin
3. Set Authentication type, e.g. Password
4. Set a password
5. Select A2 Standard (2 cores, 3.5 GB RAM) as a minimum while the code is built and blockchain is synced, then scaled back to A1 when all is set.
6. Select Option Configuration -> Endpoints -> New entry ->
 1. Endpoint: Bitcoin
 2. Private Port: 8333
 3. Protocol: TCP
 4. Public Port: 8333
 5. Click OK
7. Click OK to accept Optional config settings
8. Select Location, e.g. East US
9. Click Create
10. Wait until the VM has been created

### Attach a second 120GB empty disk (to be used for the block chain) ###

1. Select Home
2. Select the VM, e.g. BitcoinXT-2
3. Select All settings -> Disks -> Attach New ->
 1. Size (GB): 120
 2. Click OK
4. Wait until the new disk has been attached

You may want to restrict access to port 22 (SSH port) to your own IP.

Now we can SSH into the VM, using ssh, e.g. bitcoin@bitcoinxt-2.cloudapp.net

###Prerequisites

For the software to build a few dependencies must be installed first.  You can install them by the following commands.

```
sudo apt-get update
sudo apt-get install python-software-properties
sudo apt-get install build-essential libboost-all-dev automake libtool autoconf
sudo apt-get install libdb++-dev
sudo apt-get install libboost-all-dev
sudo apt-get install pkg-config
sudo apt-get install libssl-dev
sudo apt-get install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler
sudo apt-get install git
```

###Mount data drive
Format the data drive using ext3 filesystem and mount it.  The secondary drive should be named /dev/sdc
```
sudo fdisk /dev/sdc
n
p
1
w
sudo mkdir /media/data
sudo mount -t ext3 /dev/sdc 
```

###Build bitcoind
Now we will get bitcoin source from github (bitcoin brisbane fork) and compile.  Feel free to use the bitcoin/bitcoin repository if you like.
```
sudo git clone https://github.com/bitcoin/bitcoin
cd bitcoin
sudo git checkout 0.10
sudo ./autogen.sh
sudo ./configure --with-cli=no --with-gui=no --disable-wallet
sudo make 
sudo make install
```

Note, if there is an error on autogen.sh, re install the pre reqs.

###Move blocks

We now will move the blocks from the default location to the secondary drive, and create a static link.

```
sudo cp ~/.bitcoin/blocks /media/data/
sudo rm -R ~/.bitcoin/blocks
sudo ln -s /media/data/blocks blocks
```

To check this has worked perform the following
```
cd ~/.bitcoin
ls
```

"Blocks" should be in a cyan colour.

###Configure
Last step we need to add the bitcoin config file.
```
cd ~/.bitcoin
sudo nano bitcoin.conf
```

Add the following:
```
server=1
rpcuser=username
rpcpassword=xxxxxxxx
```

###Done
The daemon should now be ready to start.  Simply type bitcoind to start the node.   You should see "bitcoind starting"

###Trouble shooting

You can check the drive space by using the command df -h
Debug log files are located at ~/.bitcoin/debug.log  You can use the command sudo tail -f ~/.bitcoin/debug.log to monitor log files.

###References
https://help.ubuntu.com/community/InstallingANewHardDrive
http://askubuntu.com/questions/73160/how-do-i-find-the-amount-of-free-space-on-my-hard-drive
