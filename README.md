# Private Cloud on Kubernetes <!-- omit in toc -->

## Stage 1: Virtual Private Network
The goal of this stage was to create a VPN network between all hosts, which were supposed to be part of the PCK. For the encryption and networking software the Wireguard opensource project was chosen. 

![Diagram](VPN.drawio.png)

Among many possible options for configuration of Wireguard the one using ip link command was used. For example, the wg-quick method was interfering with kubernetes networking rules, which resulted in unsuccessful execution of kubeadm create cluster command. In the project the following prerequisites were gathered and steps were followed to obtain fully functional VPN network based on Wireguard:

### Hosts
- Kubernetes hosts: Ubuntu 20.04, x86-64 architecture bare metal machine
- Motorola phone: Android 11, ARM architecture
- VPN Server: Ubuntu 20.04, x86-64 architecture virtual machine
### VPN configuration
#### VPN server wireguard configuration

#### Kubernetes hosts wireguard configuration
#### Motorola wireguard configuration



## Stage 2: Kubernetes Cluster



## Stage 3: Hyperconvergent Storage Layer



## Stage 4: Owncloud Layer



## Stage 5: Enhancements
### Stage 5.1 HA Kubernetes
### Stage 5.2 Exposing to Internet
### Stage 5.3 Backup using QNAP NAS
### Stage 5.4 Redundant VPN
### Stage 5.5 Anonymous VPN