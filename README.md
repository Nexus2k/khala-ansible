# khala-ansible
Ansible scripts to deploy a khala mining pool.

## Setup/Component Overview

![Phala networking](https://user-images.githubusercontent.com/688267/164886024-fbada31e-fb01-4a66-9b1e-c534d1167ac5.svg)

## ToDo's for contributors

- Integrate [node-angel-v4.sh](https://github.com/zaqhack/crypto-tools/blob/main/bash/node-angel-v4.sh) as node restarter
- Add some monitoring around prb
- Update documentation

## Please Nominate or Tip Me!

* Polkadot: [12Dw4SzhsxX3fpDiLUYXm9oGbfxcbg1Peq67gc5jkkEo1TKr](https://polkadot.subscan.io/waiting/12Dw4SzhsxX3fpDiLUYXm9oGbfxcbg1Peq67gc5jkkEo1TKr) (🍁 HIGH/STAKE 🥩/HEL1 - 2.69% commission)
* Polkadot: [12bget8jJWnyjqYPiCwkXZjDh5tDBkta1WUcDYyndbXVDmQ1](https://polkadot.subscan.io/waiting/12bget8jJWnyjqYPiCwkXZjDh5tDBkta1WUcDYyndbXVDmQ1) (🍁 HIGH/STAKE 🥩/ZRH1 - 0.69% commission)
* Kusama: [DbRgw96nMQcFEFZWTLd6LSPNdh8u3NBuUDfAhDmB6UU8cJC](https://thousand-validators.kusama.network/#/leaderboard/DbRgw96nMQcFEFZWTLd6LSPNdh8u3NBuUDfAhDmB6UU8cJC) (🍁 HIGH/STAKE 🥩/HEL1 - 4.20% commission)
* Kusama: [HQuPha82sRy91zZn73XRGJVV3ernBh5HZKftUcoDT8CSUwK](https://thousand-validators.kusama.network/#/leaderboard/HQuPha82sRy91zZn73XRGJVV3ernBh5HZKftUcoDT8CSUwK) (🍁 HIGH/STAKE 🥩/ZRH1 - 10% commission)

## Getting started

Prerequisits:

*  Node: Runs the Khala/Kusama node, needs about 1T of disk space
*  PBR: Runs the Phala Bridge Runtime, needs another 1T of disk space, usually runs on the same host as the node
*  Worker: Runs the Phala pruntime, needs SGX compatibility (you can run a worker on the same machine like Node & PBR but make sure to reduce the number of CPUs used by the worker on that machine using the `cpu_cores` attribute)
*  **Lots of time**: The Node synchs for about 3-5 days, then the data_provider for another 2-3 days and then each worker/miner for another 2-3 days depending on CPU/network specs.

First copy the `hosts.ini-sample` to `hosts.ini` then:

1.  Rename the `my_*` entries to a name you like, also replace the `<IP of your server>` with the actual IP of your server
1.  Create one file per host in the inventory in the `host_vars` dir according to the sample. Only the node machine needs the `wireguard_unmanaged_peers:` entry.
1.  Run the ansible playbooks in this order:

- create_vpn_config.yml
- setup_vpn.yml - Set's up the Wireguard VPN
- all.yml - runs the following steps:
    - setup_machine.yml - Installs some base packages that are needed including sgx on the workers
    - setup_node.yml - Setups the phala/kusama node
    - setup_prb.yml - Setups the PRB stack
    - setup_worker.yml - Setups the worker/miner servers
- add_workers.yml - Adds all defined workers to the lifecycle manager (update the lifecycle_pubkey first!)

## Pool operations

You will need to check for the following, each step has to complete before you even need to bother to check for the next one, **it will take several days!!**:

1.  The node compoenent (phala-node container) needs to be fully synched. (Logs should say Idle and not Synching for both Parachain and Parentchain).
1.  Connect to the wireguard VPN to be able to access the internal IPs.
1.  The data_provider component will need to be fully synced, check by browsing to http://10.8.0.1:3000/ (monitor container), it should show the discovered data_provider -> Status it also has to go to `S_IDLE`.
1.  In the lifecycle component you will now need to add the name, pool id, mnemonic seed phrase to setup the pool, then under Workers you will need to add the workers with the same pool id (pid), the pruntime endpoint (http://10.8.0.X:8000) and the maximum stake that worker should receive.
1.  Restart the lifecycle manager container after that using `docker restart prb_lifecycle_1 -t 60`.
1.  The miners will then start synching (takes another 2-3 days!!)
1.  If anything hangs too long try to restart the `phala-node` container, it will restart the prb components automatically.

## Notes

- Monitoring is not yet covered in these ansible playbooks. Do NOT run a Khala miner w/o proper monitoring or you will get slashed.
- You should not use `root` user on the server, instead replace the `ansible_user` field in `hosts.ini` with an unpriviledged user (which has docker rights).
- [s]When the pruntime containers get's shutdown too quickly it might corrupt the pruntime workers. In that case one of the ways to recover is to copy back the checkpoint.seal.bck file but if that's also corrupt you have to start synching from scratch!![/s] - Just restart the worker in the Lifecycle UI

## Downgrade Ubuntu 22.04 LTS kernel to 5.13

```
wget https://raw.githubusercontent.com/pimlie/ubuntu-mainline-kernel.sh/master/ubuntu-mainline-kernel.sh
chmod +x ubuntu-mainline-kernel.sh

# search and find your wanted version
ubuntu-mainline-kernel.sh -r | grep 5.13

# install that version kernel
ubuntu-mainline-kernel.sh -i v5.13.19

# get all menuentries
grep 'menuentry \|submenu ' /boot/grub/grub.cfg | cut -f2 -d "'"

# change the grub configuration
vi /etc/default/grub
from: GRUB_DEFAULT=0
to: GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 5.13.19-051319-generic"

# update grub
update-grub

# reboot
reboot now

# verify
uname -r
```