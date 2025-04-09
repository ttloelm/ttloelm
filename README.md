
# Lab: Implementation of a Network Lab using VMware

## Goals
- Implement and test a virtual computer network
- Perform DHCP, Subnetting, Routing, Switching labworks

##  Virtualization tool
The setup of the machines and networks for this lab has is VMWare
## Topology
![VMWARE](https://github.com/user-attachments/assets/01c8f2a5-841c-4aa8-9020-93d49563c78a)

## Setup machines and networks
### 1. Create Bootable VMware USB Drive using RUFUS
Objective:
To prepare a USB drive that can be used to install VMware ESXi on the physical DELL 330 servers.

Tools:
RUFUS

VMware ESXi ISO image (e.g., VMware-VMvisor-Installer-8.x.x-xxxxxxx.x86_64.iso)

8GB or larger USB drive

Steps:
Insert the USB drive into your PC.

Open RUFUS, select the USB drive under "Device".

Under "Boot selection", choose the ESXi ISO file.

Partition Scheme: Choose "MBR" for BIOS or "GPT" for UEFI (depending on server BIOS settings).

Click Start, then Yes to write in ISO mode.

Why:
This is needed to boot into the ESXi installer on the DELL 330 servers, which will install the hypervisor directly to the system's disk.
![Rufus](https://github.com/user-attachments/assets/e71d06ee-c4cb-4c46-ae5e-8aa883b974dd)

### 2. Install VMware ESXi Software
Objective:
Install the ESXi hypervisor on each DELL 330 host.

Steps:
Insert the bootable USB into DELL 330 server.

Boot from USB and follow the on-screen instructions.

Choose the disk to install (e.g., local SSD).

Set the root password.

After installation, configure the management IP:

DELL 330 #1: 10.10.10.51

DELL 330 #2: 10.10.10.52

Why:
ESXi is the bare-metal hypervisor that manages virtual machines. Each host needs a unique IP for management (vCenter or direct web UI).
![putty](https://github.com/user-attachments/assets/6f79eb99-751d-451c-97bf-acc36940af0a)

### 3. Configure Network Components (vSwitch, VLAN)
Objective:
To create virtual network interfaces for the VMs and management, and to separate traffic using VLANs.

Steps on both hosts:
vSwitch0 is created by default.

Add VMkernel NIC (vmk0) with IP:

Host #1: 10.10.10.51

Host #2: 10.10.10.52

VM Network Port Groups:

Management Network – Default (No VLAN)

Port Group 10.1.1.0 (VLAN2)

Port Group 10.2.2.0 (VLAN3)

vmnic0 and vmnic1 are connected as uplinks:

vmnic0 (port 1) connected to trunk port on Cisco 2960.

vmnic1 (port 2) connects to Cisco WS2960G for iSCSI traffic.

Why:
Segmenting traffic via VLANs improves security and performance. Management, guest VM traffic, and storage (iSCSI) are isolated from each other.
![vlan](https://github.com/user-attachments/assets/599edc26-617c-4d43-b13c-03db1bd018e7)
### 4. Create Volumes on NAS (HANOI)
Objective:
Create shared storage volumes on the NAS to be used by ESXi hosts over iSCSI.

Steps:
Access NAS web interface (connected via 10.10.10.1/24).

Create iSCSI Target and LUN (volume).

Note the iSCSI Target IP and IQN.

Example IP: 10.10.10.1 (NAS HANOI)

LUN can be sized based on need (e.g., 500GB)

Why:
Shared iSCSI storage allows both ESXi hosts to access the same datastore, enabling VMotion, HA, and shared storage configurations.
![volume](https://github.com/user-attachments/assets/1546700c-4255-44a8-b4e3-a29f01bb647f)

### 5. ESXi Hosts Enable iSCSI and Add the Targets
Objective:
Configure iSCSI software adapter to connect to NAS storage.

Steps:
Go to Storage > Adapters > Add Software iSCSI Adapter.

Enable it and configure target IP: 10.10.10.1.

Add VMkernel NIC for iSCSI traffic (on VLAN 100 via vmnic1):

vmk1 IP:

Host #1: 10.10.100.1

Host #2: 10.10.100.2

Rescan the adapter and confirm detection of NAS target.

Why:
iSCSI enables block-level access over the network, allowing ESXi to treat NAS volumes like local disks for datastores.

### 6. Create the Datastore on ESXi Host
Objective:
Use the connected iSCSI LUN to create a datastore.

Steps:
Go to Storage > New Datastore.

Choose VMFS, select the iSCSI LUN.

Format and name the datastore (e.g., SharedDS1).

Repeat or share the datastore with both hosts.

Why:
Datastores are where VMs are stored. Sharing datastores between hosts allows for better flexibility and redundancy.
![Virtual](https://github.com/user-attachments/assets/5cd10e1b-0bce-44ad-9e81-5d88d987a590)

### 7. Install Guest OS (Virtual Machines)
Objective:
Deploy and run virtual machines on the ESXi hosts.

Steps:
Upload ISO images to datastore.

Create a new VM, assign to correct Port Group:

VLAN2 → 10.1.1.x network

VLAN3 → 10.2.2.x network

Choose storage from shared datastore.

Power on and install OS.

Why:
Deploying VMs is the main use of virtualization. Assigning VLANs ensures proper network segmentation.
![customer](https://github.com/user-attachments/assets/8907d310-4477-4644-9ab2-134d5737b682)
## Network Summary

| Component            | IP Address                         |Role                             |
| ----------------- | -------------------------------- |--------------------------------|
| NAS HANOI	 | 10.10.10.1	|iSCSI Storage Target|
| DELL330-1 (vmk0)	 | 10.10.10.51 |ESXi Management
| DELL330-2 (vmk0)	 |10.10.10.52	  |ESXi Management
| DELL330-1 (vmk1)	 | 10.10.100.1	 |iSCSI VMkernel (VLAN 100)
| DELL330-2 (vmk1) |10.10.100.2  |iSCSI VMkernel (VLAN 100)
| Cisco 2960	| 10.10.10.2 |Switch (Trunk for ESXi/Uplinks)
| Cisco Router | 10.10.10.1/24	 |Gateway


