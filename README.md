# khala-ansible
Ansible scripts to deploy a khala mining pool.

## Please Nominate or Tip Me!

* Polkadot: [12Dw4SzhsxX3fpDiLUYXm9oGbfxcbg1Peq67gc5jkkEo1TKr](https://polkadot.subscan.io/waiting/12Dw4SzhsxX3fpDiLUYXm9oGbfxcbg1Peq67gc5jkkEo1TKr) (🍁 HIGH/STAKE 🥩/HEL1 - 2.69% commission)
* Polkadot: [12bget8jJWnyjqYPiCwkXZjDh5tDBkta1WUcDYyndbXVDmQ1](https://polkadot.subscan.io/waiting/12bget8jJWnyjqYPiCwkXZjDh5tDBkta1WUcDYyndbXVDmQ1) (🍁 HIGH/STAKE 🥩/ZRH1 - 0.69% commission)
* Kusama: [DbRgw96nMQcFEFZWTLd6LSPNdh8u3NBuUDfAhDmB6UU8cJC](https://thousand-validators.kusama.network/#/leaderboard/DbRgw96nMQcFEFZWTLd6LSPNdh8u3NBuUDfAhDmB6UU8cJC) (🍁 HIGH/STAKE 🥩/HEL1 - 4.20% commission)
* Kusama: [HQuPha82sRy91zZn73XRGJVV3ernBh5HZKftUcoDT8CSUwK](https://thousand-validators.kusama.network/#/leaderboard/HQuPha82sRy91zZn73XRGJVV3ernBh5HZKftUcoDT8CSUwK) (🍁 HIGH/STAKE 🥩/ZRH1 - 10% commission)

## Getting started

Prerequisits:

*  Node: Runs the Khala/Kusama node, needs about 1T of disk space
*  PBR: Runs the Phala Bridge Runtime, needs another 1T of disk space, usually runs on the same host as the node
*  Worker: Runs the Phala pruntime, needs SGX compatibility

First copy the `hosts.ini-sample` to `hosts.ini` then:

1.  Rename the two `my_node_*` entries to a name you like, also replace the `<IP of your server>` with the actual IP of your server

## Deployment

If you feel adventerous you can deploy the whole server using:

```
$ ansible-playbook -i hosts.ini all.yml
```

This will execute the following roles:

* polkadot-setup
  *  Setup Docker
  *  Setup Journald
  *  Setup motd (message of the day)
* polkadot-restore-db
  *  Downloads the in the host_vars defined snapshot and unpacks it to the future `db_path`
* polkadot-validator
  *  Starts the polkadot/kusama validators as docker containers
* polkadot-claimer
  *  Copies the wallet.json & password files to the server
  *  Setups a cronjob which executes a docker container that claims the staking rewards
* polkadot-rotate-keys
  *  Rotates the session keys so you can use for the `SetSessionKeys` extrinsic

You can also run the individual roles using the `setup_*.yml` playbooks instead of `all.yml`.

### Polkadot upgrades

To upgrade to the latest Polkadot version you can simply restart the containers using the `polkadot-validator` playbook.

## Notes

Monitoring is not yet covered in these ansible playbooks. Do NOT run a Polkadot/Kusama validator w/o proper monitoring or you will get slashed.

You should not use `root` user on the server, instead replace the `ansible_user` field in `hosts.ini` with an unpriviledged user (which has docker rights).
