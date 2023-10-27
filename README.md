# pmx-cluster-tb
Proxmox cluster using 3 servers interconnected by USB4/Thunderbolt network.

This work is mainly based on prior work of [scyto](https://gist.github.com/scyto/) as described in [Proxmox Proof of Concept](https://gist.github.com/scyto/76e94832927a89d977ea989da157e9dc).
# Problem Statement
A Proxmox cluster consists of two or more hosts, each running the Proxmox Virtual Environment (VE). Virtual Machines (VM's) and Linux Containers (LXC's) are hosted by the VE.
To enable High Availability (HA) of the VE's, you can build a Cluster of VE's, so that one VE can take over the VM's and LXC's from another VE in case of issues. These issues impacting the host may include software or hardware upgrades, hardware or software failures, etc. that prevents a host from working normally.
Proxmox has the feature to build the cluster with a (dedicated) network to carry the traffic caused by the transfer of a VM from one host to another. This transfer should happen quickly, while significant amount of data must be moved. Therefor a fast, reliable and preferably dedicated synchoronization network is prefererred.
Here we propose a cluster with a dedicated synchronization network based on USB4/Thunderbolt. USB4/Thunderbolt namely supports IP (by the thunderbolt-net Linux module), so Point-to-Point IP connections can be made.
The prior work by scyto demonstrates that this works on Intel based mini personal computers (Intel NUC13). Knowing that Thunderbolt support on Intel platforms is very good, the project described here takes on the challenge to test this on AMD based mini personal computers ([Bee-link GTR7](https://www.bee-link.com/catalog/product/index?id=485)).
# Architecture
The Proxmox cluster consists of 3 identical mini pc's, each with the following configuration:
- [Bee-link GTR 7](https://www.bee-link.com/catalog/product/index?id=485) (or [GTR 7 Pro](https://www.bee-link.com/catalog/product/index?id=545)) mini pc with AMD Ryzen 7 7840HS (or AMD Ryzen 9 7940HS) processor, 32 GB DDR5 5600 RAM, 2x 2.5 Gbps Ethernet
- [Samsung 990 Pro SSD](https://www.samsung.com/nl/memory-storage/nvme-ssd/990-pro-2tb-nvme-pcie-gen-4-mz-v9p2t0bw/) 2 TB drive for boot (200 GB) and data (1800 GB)
- [Samsung 990 Pro SSD](https://www.samsung.com/nl/memory-storage/nvme-ssd/990-pro-2tb-nvme-pcie-gen-4-mz-v9p2t0bw/) 2 TB drive for backup (200 GB) and data (1800 GB)
- [Thunderbolt4 cable](https://www.aliexpress.com/item/1005004619027797.html?spm=a2g0o.order_list.order_list_main.5.49c118022JCe4g), length: 50 cm

Each of these operate as an Proxmox VE. Each server has 2x USB4 ports. These ports are used to connect to the other 2 servers, so creating a 3-node ring configuration. It does not matter in which order/which of the 2 USB4 ports connect to a particular port on that other server.

This project describes the changes that need to be made to the individual server configurations to enable an IP synchronization network.
# Proxmox Installation
Download the ISO for the Proxmox VE version 8 installation image onto a USB flash medium, such as the [Ventoy](https://www.ventoy.net/en/index.html) tool.
In the future, we want to use most of the storage space on the 2 TB Samsung SSD for cluster storage (Ceph). So when installing Proxmox on the server, select the (first) Samsung SSD but reduce the used space for Proxmox to 200 GB.
Then follow the PVE installation using default values. In my setup, I have connected one of the 2.5 Gbps ports to my home router (for internet access). The router normally manages the IP addressing by DHCP, but I have configured the router to keep the addresses between 192.168.178.1 and 192.168.178.20 outside the range used by DHCP, so I can assign a unique IP address from this range to each of the servers.
So, these servers are configured as:
1. hostname: pve1, ip address: 192.168.178.11
2. hostname: pve2, ip address: 192.168.178.12
3. hostname: pve3, ip address: 192.168.178.13
# Add Graphical User Interface to each server (optional)
Normally, Proxmox VE is installed as a headless server, i.e. is managed remotely using a web browser (port 8006). Just for convenience, we will enable the Linux GUI and install Chromium so we can manage each server locally.

