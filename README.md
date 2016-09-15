# flocksy

flocksy is an anonymous file synchronisation tool that runs in the background, 
waits for file system activity on any number of configured directories 
-- called boxes -- and synchronises these with other connected clients. 
It uses the Flock protocol for anonymous file transfers between clients. 

## Features

- Synchronise files and all changes with connected nodes
- Supports coordinated multi-way synchronisation
- Implements the Flock protocol for anonymous file synchronisation
- Watch multiple different directories (called boxes)

## Planned Features

- Automatically exchange box differences
- Interoperability with all major OS's
- Automatic syncing of USB-disks on plug-in
- Efficient Backup and conflict system

## The Flock Protocol

flocksy makes use of the Flock protocol (from which it takes its name, you know, 
Flock sync...FlockSync...flocksy...). The Flock protocol is an anonymous file 
transfer protocol aimed to frustrate traffic analysis attacks. It uses "dummy" 
traffic to hide and synchronise individual node status and file transfers. 
Briefly, when file content is transmitted, not only the original sender sends 
data but all connected nodes, masquerading the sender. The name is a reference to 
flocks of birds, where participating birds are protected by working together and 
watching each other. 

In particular, flocksy implements a (deterministic) finite-state machine that is 
a part of the Flock protocol and generated by a GSL script to produce a 
C-header-library. 

## Current Status

flocksy is currently in an alpha stage of development. While most of the behaviour 
mandated by the Flock protocol is implemented and working as intended, many parts 
of a fully working synchronisation tool are still missing, such as sufficient testing 
mechanisms. 

It is not meant for production use. At all. 

## How to build

There are currently no builds being distributed, i.e. flocksy must be compiled
from source. It uses a handful of third-party libraries:

- Boost.Filesystem (>1.55)
- Boost.Thread (>1.55)
- Boost.Test (>1.55)
- Boost.Program_options (>1.55)
- CMake
- Sodium
- zeromq4-1
- zmqpp
- JsonCpp

### Compiling on Ubuntu 16.04

#### Installing Dependencies

```
# Installing compiler environment
sudo apt-get install g++ build-essential libtool autoconf pkg-config

# Installing Boost libraries
sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-test-dev libboost-thread-dev libboost-program-options-dev

# Installing CMake
sudo apt-get install cmake

# Installing JsonCpp
sudo apt-get install libjsoncpp-dev

# Installing Sodium
sudo apt-get install libsodium-dev

# Installing git
sudo apt-get install git
```

#### Compiling ZeroMQ (with CurveZMQ) and zmqpp

```
# Compiling zeromq4-1
git clone https://github.com/zeromq/zeromq4-1.git
cd zeromq4-1/
./autogen.sh
./configure && make
sudo make install
sudo ldconfig
cd ..

# Compiling zmqpp
git clone https://github.com/zeromq/zmqpp.git
cd zmqpp/
make
sudo make install
cd ..
```

#### Compiling flocksy

```
#Compiling flocksy
git clone https://github.com/alex-dot/flocksy.git
cd flocksy/
mkdir build && cd build/
cmake ..
make
cd ../bin/
```

## How to use

To operate, the flocksy client needs at least one hostname (the complete URI 
under which the client can be found), one configured node (another flocksy 
client to connect to, which is that client's hostname), and a box name and location. 
These settings can be passed as arguments to the flocksy command-line or 
stored in a configuration file. Furthermore, a cryptographic long-term keypair 
and a keystore containing all node's public keys are required. 

### Command-line Arguments

```
Generic options:
  -h [ --help ]          Produce this help message
  -c [ --config ] arg    Use supplied config file instead of default one
  --keystore arg         Use supplied keystore file instead of default one
  --privatekey arg       Use supplied privatekey file instead of default one
  --create-private-keys  Create new keypair for hostnames

Allowed options:
  -n [ --node ] arg     Add node location to listen to (multiple arguments 
                        allowed)
  -b [ --box ] arg      Add path of a directory to watch (multiple arguments 
                        allowed)
  -p [ --hostname ] arg Add a name for this machine under which other nodes can
                        reach it (multiple arguments allowed)
```

#### Examples

`./flocksy` starts the client using the default config from `~/.flocksy`, uses 
long-term keypairs from the default `~/.ssh/flocksy_privatekeys`, and uses the 
keystore from default `~/.ssh/flocksy_keystore`. 

`./flocksy -c PATH` uses config from PATH and default keypairs and keystore. 

`./flocksy -n NODE1 -n NODE2 -p HOST -b BOX` uses HOST for the client's hostname, 
BOX for the box name and path, and connects to both NODE1 and NODE2 using default 
keypairs and keystore. 

### Creating a config file

Configurations can be permanently stored in config files. The default config 
location is `~/.flocksy`, which will be used unless another config file is 
supplied using the `-c` flag. 

An example configuration file looks like:

```
hostname = tcp://127.0.0.1:5671
node = tcp://127.0.0.1:5672
box = mainbox@/home/alex/testdir
```

Note that the current version of flock requires all configured nodes to be 
actually connected and running. Otherwise the running clients will stuck. 

### Generating Keys

To generate a new long-term keypair, simply run `./flocksy --create-new-keypair`. 
This will create the file `~/.ssh/flocksy_privatekeys` (unless it already exists
or another file location has been configured) and store for every configured 
hostname a keypair in JSON format. 

### Exchanging Keys

Public keys of all nodes must be exchanged before use of flocksy using a secure 
channel. All public keys shall be stored in a keystore file (default: 
`~/.ssh/flocksy_keystore`) in a JSON format as an array with the node's 
ID as the key and the public key as the value. 

The secure exchange of public keys is important, as adversaries are able to 
launch man-in-the-middle-attacks when exchanged insecurely. 
NO SECURITY CAN BE GUARANTEED BY INSECURE PUBKEY EXCHANCE! 

### Kickstart

For the purpose of testing locally, there are three config files stored inside 
the test directory, configuring three nodes `ipc:///tmp/node1.ipc`, 
`ipc:///tmp/node2.ipc`, and `ipc:///tmp/node3.ipc`. Additionally, there is a 
pregenerated keystore and private keys file. IPC is a supported transport that
lets you communicate between separate processes (i.e. applications) through a 
UNIX file locally on the same machine.  

You still need three folders in your `/tmp/` directory:

```
mkdir /tmp/testdir1
mkdir /tmp/testdir2
mkdir /tmp/testdir3
```

You can run the clients with the following line (replace `X` with 1, 2, or 3):

```
bin/flocksy -c test/configX --keystore test/flocksy_keystore --privatekey test/flocksy_privatekeys
```

NEVER use the sample keys in production!

## Copyright

This software is released under the GPLv3 (or later) license. See LICENSE. 
