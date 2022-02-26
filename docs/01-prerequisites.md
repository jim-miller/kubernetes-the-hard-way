# Prerequisites

## Cluster of Rasperry Pi 4s

This is a stand in for the Google compute resources.

## Ansible

This replaces the Google Cloud SDK.

## Required Hardware

This tutorial is intended to deploy Kubernetes across five Raspberry Pi 4 nodes. Two nodes will act as the control
plane, and the remaining three will serve as worker nodes. Hardware choices can be flexible, however, and you can in
theory complete this tutorial with only two nodes.

## Network Topology

OpenWRT as an edge router. 192.168.1.0/24 lan.

## Running Commands in Parallel with tmux

**Note** We may not need this since we'll be provisioning with Ansible playbooks.

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. Labs in this tutorial may require running the same commands across multiple compute instances, in those cases consider using tmux and splitting a window into multiple panes with synchronize-panes enabled to speed up the provisioning process.

> The use of tmux is optional and not required to complete this tutorial.

![tmux screenshot](images/tmux-screenshot.png)

> Enable synchronize-panes by pressing `ctrl+b` followed by `shift+:`. Next type `set synchronize-panes on` at the prompt. To disable synchronization: `set synchronize-panes off`.

Next: [Installing the Client Tools](02-client-tools.md)
