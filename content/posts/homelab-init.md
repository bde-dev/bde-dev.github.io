---
title: homelab init
date: 2025-06-06
tags:
- homelab
---
# `homelab init`

Finally got round to initializing my homelab!

It is a 4-node cluster, consisting of 1 control plane and 3 workers.

## Chosen Hardware

All 4 nodes are HP EliteDesk 800 G2 Mini PC's.

Picked them up really cheap from eBay.

## Chosen Software

I opted for `ubuntu-server-24.04 LTS` running `k3s`.

The initial OS choice was easy as I have a few years experience with `ubuntu server` at my current employer.

I have installed a few minimal playground `k3s` clusters before on `ubuntu-server` VM's, so again I thought I would stick to what I know and use `k3s`.

## Installation

I used [k3s-ansible](https://github.com/k3s-io/k3s-ansible) to bootstrap the cluster.

There are many community playbooks for installing a k3s cluster, but I generally prefer getting as close to the maintainers of the parent tool (i.e `k3s-io`) as I can.

It is likely `k3s-io` will reliably maintain *their own playbook*, rather than relying on a community maintained playbook.

I opted for the repository installation rather than through `ansible-galaxy` as to not add a dependency tied to my host `ansible-controller`.
Should the need arise, I can keep my `inventory-sample.yaml` in source control and simply re-bootstrap using the same method.

> If I find myself having to re-bootstrap often, I may bring in `k3s-ansible` as a `git submodule`.

As this is just the initial bootstrap, these are the only things that changed in my `inventory-sample.yaml`:

1. `hosts` - set to the IP addresses of my nodes
2. `ansible_user` - changed from `debian` to the primary user on my nodes
3. `k3s_version` - I set this to the latest release at the time of installation [v1.33.1+k3s1](https://github.com/k3s-io/k3s/releases/tag/v1.33.1%2Bk3s1)
4. `ansible_become_pass` - To handle passwordless SSH access from my `ansible-ontroller` to the nodes, I used `ssh-copy-id` and this `ansible_become_pass`
5. `token` - Not using `Vagrant` so commented this var out
4. `cluster_context` - set the name of the context to `homelab`

## Next Steps

I have big plans for this cluster, including `devpods`, deploying common self-hosted apps such as `linkding` and `commafeed`, and to create an environment that allows me to deploy my own `.NET` apps and expose them securely to the internet.

More details on these milestones can be found in the repo `README.md`.

I will be posting here at every milestone, and granular progress can be followed in the repo - [homelab](https://github.com/bde-dev/homelab).
