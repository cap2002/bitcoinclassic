# Host a Bitcoicn Clasic node on Ubuntu Server 14.04 LTS / Microsoft Azure #

Motivation: To create comprehensive instructions for anyone to follow to allow users to deploy a full Bitcoin Classic node permanently on using Microsoft's Azure.

If you have **any kind** of company or startup, maybe sign up through https://www.microsoft.com/bizspark/signup/ and get $150 a month per employee of free Azure time for 3 years.

### Requirements ###
1. Secondary drive large enough for the block chain growth
2. No wallet
3. No UI
4. Allow to be discovered by opening firewall
5. Good uptime

Start from the Microsoft Azure portal https://portal.azure.com

### Create an Ubuntu Server VM on Azure ###

1. Basics
 1. Select New -> Compute -> Ubuntu Server 14.04 LTS -> Create
 2. Enter Name, e.g. BitcoinClassic-1
 3. Enter User name, e.g. Satoshi
 3. Set Authentication type, e.g. Password
 4. Set a strong password
 5. Make a Resource group, e.g. Bitcoin
 6. Select Location, e.g. North Europe
 7. Click OK
2. Size
 1. Select View all
 2. Select A2 Standard (2 cores, 3.5 GB RAM) as a minimum while the code is built and blockchain is synced, then scale back to A1 when all is set.
 3. Click Select
3. Settings
 1. Keep suggested Storage, Network and Monitoring settings
 2. Click OK
4. Summary
 1. Click OK
 2. Wait for deployment to finish
5. Assign DNS name label
 1. Click the public IP address -> All settings -> Configuration
 2. Enter a DNS name label, e.g. bitcoinclassic21
 3. Click the Save icon
 
### Attach a second 120GB empty disk (to be used for the block chain) ###

1. Click Microsoft Azure in the top-left corner
2. Select Virtual machines -> BitcoinClassic-1
 1. Select Disks -> Attach new
 2. Size (GB): 120
 3. Click OK
 4. Wait until the new disk has been attached

### Open port 8333 for inbound connections  ###

1. Click All resources from the left menu
2. Select Network security group - looks like a shield
 1. Select Inbound security rules
 2. Click Add
 3. Enter name, e.g. Bitcoin
 4. Change Protocol to TCP
 4. Change Destination port range to 8333
 5. Click OK

You may want to restrict access to port 22 (SSH port) to your own IP.

Now we can SSH into the VM, e.g. satoshi@bitcoinclassic21.northeurope.cloudapp.azure.com
If connection was not made, try restarting the server.

### Prerequisites ###

For the software to build a few dependencies must be installed first.  You can install them by the following commands.

```
sudo apt-get update
sudo apt-get -y install python-software-properties
sudo apt-get -y install build-essential libboost-all-dev automake libtool autoconf
sudo apt-get -y install libdb++-dev
sudo apt-get -y install libboost-all-dev
sudo apt-get -y install pkg-config
sudo apt-get -y install libssl-dev
sudo apt-get -y install libcurl4-openssl-dev
sudo apt-get -y install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler
sudo apt-get -y install git

```

### Mount data drive ###
Format the data drive using ext3 filesystem and mount it. The secondary drive should be named /dev/sdc
```
sudo fdisk /dev/sdc
n
p
1
(Enter)
(Enter)
w
sudo mkfs.ext3 /dev/sdc1
sudo mkdir /media/data
sudo mount /dev/sdc1 /media/data
cd /media/data
sudo su -c "echo '/dev/sdc1 /media/data ext3 defaults 0 0' >> /etc/fstab"
```

### Make BitcoinXT ###
Now we will get the BitcoinXT source from GitHub and make the executables.
```
sudo git clone https://github.com/bitcoinxt/bitcoinxt
cd bitcoinxt
sudo ./autogen.sh
sudo ./configure --with-cli=yes --with-gui=no --disable-wallet
sudo make 
sudo make install
```

Note, if there is an error on autogen.sh, re install the pre reqs.

### Initial run ###
We will run bitcoind once so it generates the proper directories.

```
 bitcoind -server
```

Write down or copy the generated rpcuser and rpcpassword values.

### Configure ###
Run the following to generate the bitcoin.conf file - replace rpcpassword with generate values.

Example:
```
echo 'rpcuser=bitcoinrpc' > ~/.bitcoin/bitcoin.conf
echo 'rpcpassword=XXXXyy1yXXXyyX1XXy1XXy1XXXy1Xy1y11XyXyXyyXXy' >>  ~/.bitcoin/bitcoin.conf
echo 'server=1' >>  ~/.bitcoin/bitcoin.conf
```

Verify config-file values are in the file.
```
more ~/.bitcoin/bitcoin.conf
```

### Move blocks ###
We now move the blocks from the default location to the secondary drive, and create a static link.

```
cd /
sudo rsync -a ~/.bitcoin/blocks /media/data/
sudo rm -R ~/.bitcoin/blocks
ln -s /media/data/blocks blocks
```

To check this has worked perform the following.
```
cd ~/.bitcoin
ls -al
```

"blocks" should be in a cyan colour and look like this: blocks -> /media/data/blocks
Node that it's the current user, bitcoin, that has file access.

### Done ###
The daemon should now be ready to start.

Type the following to start the node.
```
bitcoind -daemon
```

You should see "bitcoind starting"

### Test ###
Type the following to run a small test
```
bitcoin-cli getinfo
```

The output should be something like this
```javascript
{
    "version" : 110000,
    "protocolversion" : 70010,
    "blocks" : 90012,
    "timeoffset" : -1,
    "connections" : 8,
    "proxy" : "",
    "difficulty" : 3091.73689041,
    "testnet" : false,
    "relayfee" : 0.00001000,
    "errors" : ""
}
```

Also, go to https://getaddr.bitnodes.io/ and type in the node network address, e.g. bitcoinxt-2.cloudapp.net, and click "CHECK NODE". Something like the following should show to verify you are running BitcoinXT:

`137.135.101.182:8333 /Bitcoin XT:0.11.0/`

### Trouble shooting ###

You can check the drive space by using this command
```
df -h
```

Debug log files are located at ~/.bitcoin/debug.log
You can use the command this command to monitor log files:
```
sudo tail -f ~/.bitcoin/debug.log
```
