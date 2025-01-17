# Debian Package Guide
Author: Jason (jagerman) 

Source: [https://deb.imaginary.stream/](https://deb.imaginary.stream/)

This repository contains debian/ubuntu builds of the core loki tools (lokid, cli/rpc wallets,
blockchain tools) for Debian and Ubuntu.

## Requirements

One of:

- Debian 9 ("stretch")
- Debian 10 ("buster")
- Debian unstable ("sid")
- Ubuntu 16.04 ("xenial")
- Ubuntu 18.04 ("bionic")
- Ubuntu 19.04 ("disco")


## Express Guide

Start a new service node by running these four commands:

```
curl -s https://deb.imaginary.stream/public.gpg | sudo apt-key add -

echo "deb https://deb.imaginary.stream $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/imaginary.stream.list

apt update

apt install loki-service-node
```

The services will run via systemd as `loki-node.service` and `loki-storage-server.service`.

## Full Guide

### 1: Initial Repository Setup

You only need to do this step the first time you want to set up the repository; when you've done it
once, the repository will automatically update whenever you fetch new system updates.

To add the apt repository run the following command, replacing `<DISTRO>` in the second line with the
appropriate value (see below).

The following command installed Jagermans public key used to sign the Binaries.
```
curl -s https://deb.imaginary.stream/public.gpg | sudo apt-key add -
```

The second command tells `apt` where to find the packages and needs you to replace `<DISTRO>` with your your distro.

To find your `<Distro>` run the following command: `lsb_release -sc`

Alternatively your `<Distro>` can be found by using the following list:

- sid      (Debian testing/unstable)
- stretch  (Debian 9)
- buster   (Debian 10)
- xenial   (Ubuntu 16.04)
- bionic   (Ubuntu 18.04)
- disco    (Ubuntu 19.04)

```
echo "deb https://deb.imaginary.stream <DISTRO> main" | sudo tee /etc/apt/sources.list.d/imaginary.stream.list
```

Then resync your package repositories with:
```
sudo apt update
```

Now you can install one or more of the following packages as desired:

| **Package**                 | **Description**                                                                                            |
|-------------------------|--------------------------------------------------------------------------------------------------------|
| `loki-service-node`     | Metapackage that does everything you need for a running service node.                                  |
| `lokid`                 | The loki daemon (automatically pulled in by `loki-service-node`).                                      |
| `loki-storage-server`   | The loki storage server, required for a service node (automatically pulled in by `loki-service-node`). |
| `loki-wallet-cli`       | The command-line wallet.                                                                               |
| `loki-wallet-rpc`       | The rpc wallet (for script-based wallet interaction such as a pool would need).                        |
| `loki-blockchain-tools` | the various loki-blockchain-* commands for advanced blockchain management.                             |


There are also a couple of `libloki-core*` packages containing the shared library code, but these will
be installed automatically as needed.  

There is also a `libloki-core-dev` package containing the loki
headers, but it's highly unlikely that you will need that for common use.

#### Installing a Package

To install a package run the following command replacing <package> with one of the packages available above:

```
sudo apt install <package>
```

For example: To install `loki-service-node` package run the following command:
```
sudo apt install loki-service-node
```

### 2. Loki Service Node Operation

Running a service node requires multiple packages (lokid and loki-storage-server) with synchronized
configuration between them.  There are two ways to approach this:

#### **Automatic**:  

Install the `loki-service-node` package.

```
sudo apt install loki-service-node
```
This will detect your public IP (or allow you to enter it yourself) and automatically update the loki.conf configuration file with the necessary additional settings to run a service node.

#### **Manually**:  

Install both `lokid` and `loki-storage-server` packages

```
sudo apt install lokid
```

```
sudo apt install loki-storage-server
```

Then edit the configuration in `/etc/loki/loki.conf` and `/etc/loki/storage.conf`.  

If you want to change some of the lokid settings you can add the settings to `/etc/loki/loki.conf`.

Restart both after configuration updates using:

```
systemctl restart loki-node loki-storage-server
```

---

#### Adding flags to config file.

Note that you can add any of lokid's command-line arguments as settings in this file rather than
needing to change the service file.  For example, if you wanted to run lokid as
`lokid --p2p-bind-ip 1.2.3.4 --p2p-bind-port=22222 --restricted-rpc` then you would add:

```
p2p-bind-ip=1.2.3.4
p2p-bind-port=22222
restricted-rpc=1
```

into `/etc/loki/loki.conf` and then restart lokid using:

```
sudo systemctl restart loki-node
```

---

### 3. Interacting with the running lokid

If you run the lokid binary with a command, it forwards this command to the running lokid. So, for example, to get the current lokid status you can run (note that sudo is not required!):
```
lokid status

lokid print_sn_status
```

To prepare a service node registration run the following command:

```
lokid prepare_registration
```

The terminal will show the next steps to conduct to have your Service Node staked and thus receiving rewards.

To see the outputs of your node you can run the following command:
```
journalctl -u loki-node -af
```
This is useful to see if your node is syncing with the blockchain.

For a full list of supported commands run:

```
lokid help
```

For interacting with a running testnet node, add `--testnet` into the command, such as:

```
lokid --testnet status
lokid --testnet print_sn_status
lokid --testnet prepare_registration
```

## Additional/Optional: 

### Upgrading

When a new release is available upgrading it as simple as syncing with the repository:

```
sudo apt update
```

Then install any updates using:

```
sudo apt upgrade
```

> Note that this will install both updated lokid packages *and* any available system updates (this is generally a good thing!).  

During the upgrade, lokid (both mainnet and testnet) will be restarted if they are currently running to switch to the updated lokid.

If for some reason you want to install *only* updated loki package upgrades but not other system packages then instead of the `sudo apt upgrade` you can use:

```
sudo apt install loki-storage-server lokid loki-wallet-cli
```

---

### Installing a non-service-node lokid and cli wallet.

To install a non-service-node lokid and the cli wallet:

```
sudo apt install lokid loki-wallet-cli
```

Once installed the binaries will be in your path, so you can simply run `loki-wallet-cli` as an
ordinary user to use the command-line wallet.

#### Non-service-node Loki operation

Installing the lokid package sets up a `loki-node` systemd service that runs an ordinary loki node,
but not a service node.  This node can be used, for example, for wallet synchronization.  If you
don't want the node to run you can stop it after installation using:

```
sudo systemctl stop loki-node
```

If you want to disable it entirely (so that it doesn't start up automatically at boot) you can use:

```
sudo systemctl disable --now loki-node
```

If you later want to re-enable it you would use:

```
sudo systemctl enable --now loki-node
```

> Remove the `--now` if you want to reenable it but not actually try starting it.

For advanced users, lokid creates a `_loki` user and stores all the files (blockchain, service node
key, etc.) owned by this user in `/var/lib/loki`.

---

### Running a Testnet Node

The lokid package also installs a second systemd service, loki-testnet-node, for easily running a testnet node (either at the same time or instead of the mainnet node).  To enable and start this, simply run:

```
sudo systemctl enable --now loki-testnet-node
```

This will start it and configure systemd to automatically start it when the system boots.

The `loki-service-node` package also updates the testnet.conf file to run as a (testnet) service
node; in order to actually run it you need to activate and start both the loki-testnet-node.service
(as above) and also the loki-testnet-storage-server.service:

    sudo systemctl enable --now loki-testnet-node
    sudo systemctl enable --now loki-testnet-storage-server

---

## Reporting issues

Contact me via @jagerman42 (Telegram) or @jagerman#6841 (in the Loki Discord).