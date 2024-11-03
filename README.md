# Build IOG specified version of libsecp256k1 as a deb package
This document outlines _one_ way to build the IOG specified version of libsecp256k1 (needed for compiling the cardano-node software) into a Debian package.  Building this library into a deb package makes it easier to install on any Debian or Ubuntu system and have the package management system take care of everything.

This is not an official deb package and it does not do things the official Debian/Ubuntu way.  Furthermore, this deb package does not deliver the proper copyright documents or any manuals.

This deb package will install the software in the following standard locations:

* Libraries get placed in /usr/lib/

## Setup your build environment
I usually create a virtual machine for building software and do everything on it.  This allows me to keep everything separated from my desktop PC.

I also create a separate user ('builder') for building.  This is advisable, even if you choose to build on your desktop PC, so that the installation of any local compilers, or any specific configurations, won't interfere with anything in your normal user account.
```
sudo adduser --gecos '' --disabled-password builder
```

Install build dependencies and some extra requirements
```
sudo apt install build-essential fakeroot devscripts debhelper autoconf-archive d-shlibs pkg-kde-tools git curl automake pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make jq libncursesw5 libtool autoconf libncurses-dev libncurses5 libnuma-dev
```

Recommend install llvm and set cc and c++ to use clang
```
sudo apt install llvm clang; \
sudo update-alternatives --install /usr/bin/cc cc /usr/bin/clang 100; \
sudo update-alternatives --install /usr/bin/c++ c++ /usr/bin/clang++ 100
```
Ubuntu users might need to skip the previous optional step if their Ubuntu version of clang doesn't support optimization flag '-ffat-lto-objects'.  Revert the previous changes with:
```
sudo apt purge llvm clang; \
sudo apt autoremove;
```

Switch to your builder account
```
sudo su - builder
```

The rest of this document assumes you are using your 'builder' account:  
builder@build:~$

IOG specified SECP256K1_VERSION depends on CARDANO_NODE_VERSION so set this value appropriately to what you want.  This will set the value to the latest version released for mainnet.
```
CARDANO_NODE_VERSION="$(curl -s https://api.github.com/repos/intersectMBO/cardano-node/releases/latest | jq -r .tag_name)"; \
echo "Latest CARDANO_NODE_VERSION is: ${CARDANO_NODE_VERSION}";
```

The following sequence of commands will remove and recreate the "${HOME}/src/libsecp256k1-iog" directory.  Familiarise yourself with the commands before running them.  You can simply copy and paste the entire list of commands below into a bash terminal to run them in sequence once you have set the CARDANO_NODE_VERSION variable.
```
deb_build_instructions_repo='https://github.com/TerminadaPool/libsecp256k1-iog-debian.git'; \

IOHKNIX_VERSION=$(curl https://raw.githubusercontent.com/IntersectMBO/cardano-node/$CARDANO_NODE_VERSION/flake.lock | jq -r '.nodes.iohkNix.locked.rev'); \
echo "iohk-nix version: $IOHKNIX_VERSION"; \
SECP256K1_VERSION=$(curl https://raw.githubusercontent.com/input-output-hk/iohk-nix/$IOHKNIX_VERSION/flake.lock | jq -r '.nodes.secp256k1.original.ref'); \
SECP256K1_VERSION="${SECP256K1_VERSION#v}"; \
echo "Using secp256k1 version: ${SECP256K1_VERSION}"; \

package='libsecp256k1-iog'; \
basedir="${HOME}/src/${package}"; \

mkdir -p "${basedir}"; \
cd "${basedir}"; \
rm -rf "${package}-${SECP256K1_VERSION#v}"; \

git clone --depth 1 --branch "v${SECP256K1_VERSION}" https://github.com/bitcoin-core/secp256k1 "${package}-${SECP256K1_VERSION}"; \
cd "${package}-${SECP256K1_VERSION}"; \

git clone "${deb_build_instructions_repo}" debian; \

debuild -us -uc -b; \

unset deb_build_instructions_repo IOHKNIX_VERSION SECP256K1_VERSION package basedir;
```

Your debs will be produced in the parent directory: "${HOME}/src/libsecp256k1-iog/".  They will be named something like:  
* libsecp256k1-dev-iog_0.3.2_amd64.deb
* libsecp256k1-2-iog_0.3.2_amd64.deb

Put these files in your own local debian repository and install them on every machine required using apt.

## Notes
* libsecp256k1-dev-iog package is configured to conflict with the standard Debian/Ubuntu libsecp256k1-dev package.
* Similarly, libsecp256k1-2-iog is configured to conflict with the standard libsecp256k1-2 package.
* Thus installing libsecp256k1-dev-iog and/or libsecp256k1-2-iog will cause the corresponding standard package to be removed.

See: 'control' file for where this is configured.

Expect to see the following lintian errors at the end of the build process:  
> E: libsecp256k1-2-iog: no-copyright-file  
> E: libsecp256k1-dev-iog: no-copyright-file  
> W: libsecp256k1-2-iog: package-name-doesnt-match-sonames libsecp256k1-2  


### References
See: https://github.com/input-output-hk/cardano-node-wiki/blob/main/docs/getting-started/install.md

