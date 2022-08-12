# Packaging libsecp256k1 (IOG version) as a debian package
This document outlines one way to compile the IOG verson of libsecp256k1 into a debian package.
This provides a way to easily install the IOG version of libsecp256k1-dev onto any stable debian system and have the package management system take care of everything.
The IOG version of libsecp256k1 is required for compiling cardano-node version 1.35.0 onwards.

This is not an official debian package and it doesn't do things the official debian way.  In particular, it is not possible to use the official debian stable version of haskell to compile the cardano-node.  Furthermore, this deb package will not deliver the proper copyright documents or any manuals.

The deb package produced will install things in the following standard locations:
* Libraries (libsecp256k1.so.0, libsecp256k1.a, libsecp256k1.la) get placed in /usr/lib/

# How to make your own libsecp256k1-0-iog and libsecp256k1-dev deb packages
## Setup your build environment
Install build dependencies and some extra requirments  
(as root)  
```
apt install build-essential fakeroot devscripts debhelper git curl automake build-essential pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make g++ jq libncursesw5 libtool autoconf llvm llvm-dev libnuma-dev libncurses-dev libncurses5
```
Some of those may only be required for building cardano-node.  But if you are trying to build the IOG version of libsecp256k1 then you obviously want this to compile cardano-node anyway.

## Create a separate user ('builder') for building
This is advisable so that the installation of GHC and its configuration won't interfere with anything in your normal user account.  
(as root)  
```
adduser --gecos '' --disabled-password builder
```
Nb. Password logins disabled.  
Now you can use 'builder' user to do everything.  

#### Switch user to your builder account  
(as root)  
```
su - builder
```

****
## The rest of this document assumes you are using your 'builder' account.
The following commands are all run from this 'builder' user account.

Where there is a number of commands, each line ends with a backslash continuation sequence.  The sequence of commands can then be simply copied and pasted into a terminal to save typing.
****

# Build the libsecp256k1-0 and libsecp256k1-dev debs
This sequence of commands will completely remove and recreate the '~/src/libsecp256k1' directory.  
Familiarise yourself with the following commands first.  
Then you can simply copy and paste them all into a terminal and press enter.  
```
rm -rf ${HOME}/src/libsecp256k1; \
mkdir -p ${HOME}/src/libsecp256k1; \
cd ${HOME}/src/libsecp256k1; \
git clone https://github.com/bitcoin-core/secp256k1; \
cd secp256k1; \
git checkout ac83be33; \

version=0.1; \
cd ..; \
mv secp256k1 libsecp256k1-iog-${version}; \
tar cvzf libsecp256k1-iog_${version}.orig.tar.gz libsecp256k1-iog-${version}; \
cd libsecp256k1-iog-${version}; \
unset version; \
git clone https://github.com/TerminadaPool/libsecp256k1-iog-debian.git debian; \
debuild -us -uc;
```

Your debs will be produced in the parent directory: ~/src/libsecp256k1/  
And named something like: libsecp256k1-0-iog_0.1-1_amd64.deb and libsecp256k1-dev-iog_0.1-1_amd64.deb

****
****
