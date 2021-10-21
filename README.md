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
1. Execute all commands as sudo: `sudo su` 
2. Install Wireguard: `apt-get update && apt-get install wireguard`
3. Generate keys: `wg genkey | tee privatekey | wg pubkey > publickey`
4. Create a configuration file: `nano /etc/wireguard/wg0.conf` and fill it in with the following content:
    ```
    [Interface]
    Address = 10.10.10.1/24
    ListenPort = 51820
    PrivateKey = <content-of-generated-privatekey-file>
    PostUp = iptables -A FORWARD -i wg0 -j ACCEPT;  iptables -t nat -A POSTROUTING -o eth0 -j    MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT;     ip6tables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    PostDown = iptables -D FORWARD -i wg0 -j ACCEPT;    iptables -t nat -D POSTROUTING -o eth0 -j  MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT;   ip6tables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

    [Peer]
    PublicKey =     <content-of-generated-publickey-file-on-the-client>
    AllowedIPs = 10.10.10.11/32

    [Peer]
    PublicKey =     <content-of-generated-publickey-file-on-the-client>
    AllowedIPs = 10.10.10.12/32
    ```
    Depending on the version of wireguard it might be   advisable to remove the 
    ```
    Address = ...
    ``` 
    line from the configuration file.

5. Power on the wireguard interface: `ip link add dev wg0 type wireguard && ip addr add 10.10.10.1/24 dev wg0 && wg addconf wg0 /etc/wireguard/wg0.conf && ip link set wg0 up`

6. To change the configuration and reapply it, execute: `ip link set wg0 down && wg addconf wg0 /etc/wireguard/wg0.conf && ip link set wg0 up`

#### Kubernetes hosts wireguard configuration
1. Execute all commands as sudo: `sudo su` 
2. Install Wireguard: `apt-get update && apt-get install wireguard`
3. Generate keys: `wg genkey | tee privatekey | wg pubkey > publickey`
4. Create a configuration file: `nano /etc/wireguard/wg0.conf` and fill it in with the following content:
    ```
    [Interface]
    PrivateKey = <content-of-generated-privatekey-file>
    ListenPort = 51820

    [Peer]
    PublicKey = <content-of-VPN-server-publickey>
    Endpoint = <public-IP-of-VPN-server>:51820
    AllowedIPs = 0.0.0.0/0
    ```
5. Power on the wireguard interface: `ip link add dev wg0 type wireguard && ip addr add 10.10.10.1/24 dev wg0 && wg addconf wg0 /etc/wireguard/wg0.conf && ip link set wg0 up`

6. To change the configuration and reapply it, execute: `ip link set wg0 down && wg addconf wg0 /etc/wireguard/wg0.conf && ip link set wg0 up`

#### Set up firewall
Execute on each host IN THIS ORDER:
1. `ufw disable`
2. `ufw allow 22` - this enables the ssh connections
3. `ufw allow 51820` - replace `51820` in case some other port is used by wireguard on the host
4. `ufw enable`

## Stage 2: Kubernetes Cluster
The installation path of the Kubernetes cluster is based on the official kubeadm path by Kubernetes:
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

The major adjustment here is using CRI-O as a container runtime, instead of commonly used docker. The attempt to set up the k8s cluster using docker was done but it appeared to be unsuccessful. The suspected reason of failure were the collisions in IPtables between docker and wireguard. Using CRI-O as a container runtime made the attempt successfull, so the docker-path was not continued anymore.

1. Complete the networking pre-requisites: 
    - On the master nodes execute: 
    `ufw disable && ufw allow 6443 && ufw allow     2379:2380/tcp && ufw allow 10250 && ufw allow 10259 &   & ufw allow 10257 && ufw enable` 
    - On the worker nodes execute: `ufw disable && ufw allow 10250 && ufw allow 30000:32767/tcp && ufw enable`
2. On all nodes load the br_netfilter: `modprobe br_netfilter`
3. On all nodes create a new file: `nano /etc/modules-load.d/k8s.conf` and insert the following content:
    ```
    br_netfilter
    ```
4.	On all nodes create a new file: `nano /etc/sysctl.d/k8s.conf`, and insert the following content:
    ```
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    ```
5. Execute: `sysctl --system` to load new configuartions. Verify there is still VPN and Internet connection on the Kubernetes nodes.

6. Disable swap: `swapoff -a`

7. On all hosts create/edit a file: `nano /etc/default/kubelet`, and put this content into it:
    ```
    KUBELET_EXTRA_ARGS=--node-ip=<IP-address-of-wireguard-interface-on-the-host>
    ```
    This network interface will be used by kubelet to deploy the Kubernetes node.

8. Install kubeadm: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl

9. Make sure there are compatible versions of kubeadm and kubelet on all hosts. Also, make sure the version of CRI-O is set accordingly.





## Stage 3: Hyperconvergent Storage Layer



## Stage 4: Owncloud Layer



## Stage 5: Enhancements
### Stage 5.1 HA Kubernetes
### Stage 5.2 Exposing to Internet
### Stage 5.3 Backup using QNAP NAS
### Stage 5.4 Redundant VPN
### Stage 5.5 Anonymous VPN