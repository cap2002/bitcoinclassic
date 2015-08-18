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
6. Select Option Configuration -> Network -> IP adresses ->
 1. Instance IP address: On
 2. Click OK
 3. Click OK
7. Select Endpoints -> New entry ->
 1. Endpoint: Bitcoin
 2. Private Port: 8333
 3. Protocol: TCP
 4. Public Port: 8333
 5. Click OK
8. Click OK to accept Optional config settings
9. Select Location, e.g. East US
10. Click Create
11. Wait until the VM has been created

### Attach a second 120GB empty disk (to be used for the block chain) ###

1. Select Home
2. Select the VM, e.g. BitcoinXT-2
3. Select All settings -> Disks -> Attach New ->
 1. Size (GB): 120
 2. Click OK
4. Wait until the new disk has been attached

You may want to restrict access to port 22 (SSH port) to your own IP.

Now we can SSH into the VM, e.g. bitcoin@bitcoinxt-2.cloudapp.net. If connection was not made, try restarting the server.

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
