Here is how you install and use IPFS to transfer big files in P2P network.

# Install
Get IPFS from the homepage:
```
wget https://dist.ipfs.io/go-ipfs/v0.10.0/go-ipfs_v0.10.0_linux-amd64.tar.gz
```
Extract:
```
tar -xvzf go-ipfs_v0.10.0_linux-amd64.tar.gz
```
Then install it:
```
cd go-ipfs
sudo bash install.sh
```
# Usage
Before using IPFS for the first time, we need to initialize the repository with the `ipfs init` command:
```
ipfs init
```
```
generating ED25519 keypair...done
peer identity: 12D3KooWKpTagdctMELCus9V41CgwiXDxuuy7rXxHgfruYykEmi9
initializing IPFS node at /home/masterpi/.ipfs
to get started, enter:

        ipfs cat /ipfs/QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc/readme

```
If you run the command from the output, you should see this:
```
Hello and Welcome to IPFS!

██╗██████╗ ███████╗███████╗
██║██╔══██╗██╔════╝██╔════╝
██║██████╔╝█████╗  ███████╗
██║██╔═══╝ ██╔══╝  ╚════██║
██║██║     ██║     ███████║
╚═╝╚═╝     ╚═╝     ╚══════╝

If you're seeing this, you have successfully installed
IPFS and are now interfacing with the ipfs merkledag!

 -------------------------------------------------------
| Warning:                                              |
|   This is alpha software. Use at your own discretion! |
|   Much is missing or lacking polish. There are bugs.  |
|   Not yet secure. Read the security notes for more.   |
 -------------------------------------------------------

Check out some of the other files in this directory:

  ./about
  ./help
  ./quick-start     <-- usage examples
  ./readme          <-- this file
  ./security-notes
```
Take a look at the config file, change ports, and anything you need:
```
ipfs config show
ipfs config edit
```
Then you will have to start ipfs:
```
screen -S ipfs #Create a new screen
ipfs daemon
```
then press `Ctrl`+`A`+`D` to exit the screen.

**Add (Upload) the file you want to ipfs** with:
```
ipfs add PATH/TO/FILE
```
It will create a new hash for your file like this `QmWrZAyfLFZM4GWCiRrRh4ZXAiFQFzuLxABcFstAxWD4Sx`, and you will have to use this hash to download this file.

From now, you can download the file normally with curl:
```
curl gateway/ipfs/$FILE_HASH
```
with the gateway from `ipfs config show`, and `$FILE_HASH` is the hash of the file you added above
